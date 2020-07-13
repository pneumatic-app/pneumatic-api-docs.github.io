<p align="center">
  <img src="https://raw.githubusercontent.com/slatedocs/img/main/logo-slate.png" alt="Slate: API Documentation Generator" width="226">
</p>

We are using Slate to generate our API documentation.

https://api-docs.pneumatic.app is running on GitHub Pages from /docs directory.


### Current Flow
* Run `bundle exec middleman build` to build actual version of docs.
* Push to this origin
* API docs will be updated soon.


### Future Flow
* Automatically built Swagger on each API change
* Swagger converts into markdown
* Slate converts markdown into HTML
* Actual version of API docs is running 
