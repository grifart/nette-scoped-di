# Nette DI Scope

Split your app dependency injection container into well-defined scopes and enforce relationships between them.

## Example

You have *model* where you want to expose access **only to** few facades. So you export them from *model* container into *app* container and the *app* then can **only access** those exported services. This means that *app* can no longer ignore ACL rules or directly access database.



## On the edge between monolith and microservices

This approach stays in the middle of monolith and microservices.

From architectonical perspective it is equavalent to microservices approach. There is clear boundary and interface between *model* and *app*.

Technically you save tons of hours of maintainace cost of managing microservices. Because there is still one app, but with many independent modules. They have clear boundaries and explicitly defined dependencies. And when you break this boundary, you get an exception. 

So if needed you can easily turn already defined modules into microservices later. Only part that is missing here is a serialization & networking stack. This can be easily solved by adding RPC layer between modules that are extracted into microservices.


**More resources:**

[Live stream of Jan Tvrdik's talk](https://www.facebook.com/pehapkari/videos/1604024302980707/) (Czech only)









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

