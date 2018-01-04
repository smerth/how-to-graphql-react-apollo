# Deploy to Heroku

 Heroku is a popular web service which allows us to deploy our site just by pushing our code up to a Heroku repository. 



## Check if Heroku CLI is installed

Run either of these @ terminal

```bash
heroku apps --all
```

or 

```bash
heroku help
```



## Install Heroku

First install Homebrew then:

@ terminal

```bash
brew install heroku
```



## Create your Heroku Account

Do this at:  [Heroku](https://www.heroku.com)



## Sign into Heroku

@ terminal
```bash
heroku login
```

follow the prompts



## Create our Heroku App

One of the neat things about us using `create-react-app` to build our starter project was that it now allows us to use the `create-react-app-buildpack` for Heroku, which is publicly available here.

[create-react-app-buildpack for Heroku](https://github.com/mars/create-react-app-buildpack)

A buildpack is essentially a piece of code that is going to tell Heroku how to compile, configure, and start the code that we deploy to our Heroku repository so that the code on Heroku runs like the code that we're running in our local dev environment. 

The way we do that is 

```bash
heroku create APP-NAME --buildpack https://github.com/mars/create-react-app-buildpack.git
```

Note the APP-NAME must be unique through-out the Heroku site.  So namespacing your apps with a common unique prefix might be a good idea.



## Add environment variables if necessary

```bash
heroku config:set VARIABLE_NAME=somevariable --app HEROKU_PROJECT_NAME
```



## Add a static.json file

```bash
touch static.json
```

@ static.json

```json
{
  "root": "build/",
  "clean_urls": false,
  "routes": {
    "/**": "index.html"
  }
}
```



## Commit changes

```bash
git add . && git commit
```



## Push changes to  Heroku

This is the equivalent of deploying

```bash
git push heroku master
```



## Launch the app in browser

```bash
heroku open
```



