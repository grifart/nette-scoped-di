# Nette Scoped Depenedency Injection Container

Split your app dependency injection container into well-defined scopes and enforce relationships between them.



## On the edge between monolith and microservices

This approach stays in the middle of monolith and microservices.

From architectonical perspective it is equavalent to microservices approach. There is clear boundary and interface between *model* and *app*.

Technically you save tons of hours of maintainace cost of managing microservices. Because there is still one app, but with many independent modules. They have clear boundaries and explicitly defined dependencies. And when you break this boundary, you get an exception. 

So if needed you can easily turn already defined modules into microservices later. Only part that is missing here is a serialization & networking stack. This can be easily solved by adding RPC layer between modules that are extracted into microservices.


**More resources:**

- [Jan Kuchar: Why you need to have more then one DI](https://youtu.be/OmjQ39HWiy0?feature=shared) (english)
- [Live stream of Jan Tvrdik's talk](https://www.facebook.com/pehapkari/videos/1604024302980707/) (Czech only)



## Example

You have *a model* where you want to expose access **only to** a few facades. So you export them from the *model* container into the *app* container, and the *app* then can **only access** those exported services. This means that *app* can no longer ignore ACL rules or directly access a database.

### Registering the model into a user DI container

For the model container, you must create a DI extension to register all the services exported from the *model* to the user's application (UI/API/CLI/...) part of the app.

```php
<?php

declare(strict_types=1);

namespace App\Infrastructure;

use Mangoweb\NetteDIScope\ScopeExtension;
use Nette\Configurator;

final class ModelExtension extends ScopeExtension
{
	public static function getTagName(): string
	{
		// Every service marketed with an `exported` tag will be available in every *user* DI container
		return 'exported';
	}

	protected function createInnerConfigurator(): Configurator // @phpstan-ignore-line
	{
		$configurator = parent::createInnerConfigurator();
		$configurator->addConfig(__DIR__ . '/../container.neon'); // the configration entrypoint of the *model* container
		$configurator->addConfig(__DIR__ . '/../../config/config.local.neon'); // parameters that are needed for the *model* container
		$configurator->addStaticParameters([ // provide the %modelDir% parameter so the model is self-aware of where it lays
			'modelDir' => __DIR__ . '/../'
		]);
		return $configurator;
	}
}

```

In the *user* DI container, register this newly created extension:

```yml
# the app-container.neon
# import public services from model-level container
# see Model/**/*.neon files for more info
extensions:
	model: Ivy\Infrastructure\ModelExtension

```

## *Exporting a service* from the model container

```yml
# model-container.neon
services:
	-
		factory: Ivy\Survey\SurveyFacade(
			# dependencies...
		)
		tags: [exported] # This makes it available for the outer container
```



## *Requiring a dependency* of the model container

When the model container needs a dependency (EmailSender), it can require it by defining the service *torso*.


```php
namespace App\Model\Infrastructure\Email;
interface EmailSender {
    function send(TheMessage $message): void;
}
```

Then tell the DI that the outer container **MUST** provide the `EmailSender.`

```yml
# model-container.neon

services:
	-
		type: App\Model\Infrastructure\Email\EmailSender
		factory: @outerContainer::getByType(Ivy\Infrastructure\Email\EmailSender)

```


The outer container then registers the service, and you are done.

```yml
services:
    - MyEmailSender # implements App\Model\Infrastructure\Email\EmailSender
```



# Danger zone

Sometimes, you need to break the pattern of being insulated from the model. See [danger zone](DANGERZONE.md).





# Contributing

This repository is fork of [mangoweb-backend/nette-di-scope](https://github.com/mangoweb-backend/nette-di-scope).

Primary development place is [gitlab.grifart.cz](https://gitlab.grifart.cz/grifart/nette-di-scope). `master` is automatically replicated back to [github:grifart/nette-di-scope](https://github.com/grifart/nette-di-scope).

## Submitting contribution into fork

### Internal contributions

Use GitLab and open merge request, target: `master`. After merge it will be automatically propagated to GitHub.

### External contributions

**External developers:** Use GitHub, open pull request targeting [our master `master`](https://github.com/grifart/nette-di-scope/tree/master).

**Internal reviewer and maintainer of pull request:**

- fetch changes to local repo
- then merge into local `master`
- push local master into `gitlab/master` (this will be automatically propagated to GitHub)
- remove branches that are not needed anymore

## Recommended git remotes setup

```bash
git remote add origin https://gitlab.grifart.cz/grifart/nette-di-scope.git
git remote add github-grifart https://github.com/grifart/nette-di-scope.git
git remote add github-mangoweb https://github.com/mangoweb-backend/nette-di-scope.git
```

