# Deploy to Heroku











\- [Instructor] I know we've been doing a lot of set up work, and you're probably anxious to start building some of the sites functionality, but before we get to that, we're going to see how we can set up our application so that it can be easily deployed up to the real live internet, via Heroku. Heroku is a popular web service which allows us to deploy our site just by pushing our code up to a Heroku repository. Before we get to that though, we of course have to make a few tweaks. If you're not familiar with Heroku, I encourage you to create an account and poke around a bit on the site. You should also make sure you have the Heroku CLI tool installed globally on your local machine, which we did in the installing local dependencies video.

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

First install Homebrew

Next @ terminal

```bash
brew install heroku
```



## Create your Heroku Account

Do this at:  [Heroku](https://www.heroku.com)



## Sign into Heroku

Before we can deploy from the command line however, we need to make sure we're locked in. I'm going to navigate to my command line. We're going to type in the command Heroku login.

I'll put in my email. And my password. I have two-factor setup, so I'm going to enter that in now. And great. Now that we're logged into Heroku, we're going to create our app.  







## Create our Heroku App

One of the neat things about us using create react app to build our starter project was that it now allows us to use the create react app buildpack for Heroku, which is publicly available here.



[create-react-app-buildpack for Heroku](https://github.com/mars/create-react-app-buildpack)







A buildpack is essentially a piece of code that is going to tell Heroku how to compile, configure,and start the code that we deploy to our Heroku repository so that the code on Heroku runs like the code that we're running in our local dev environment. 



The way we do that is we say Heroku create and now we say our app name. We're going to call ours tic tac turing instructional. A note here about Heroku app names and the Heroku CLI.

Heroku app names must be globally unique, which means that your Heroku app must have a different name than everyone else's Heroku app. So, I'm going to name mine X, and that means that you can't give it the same name. If I name mine tic tac turing instructional, you can name yours whatever you want, as long as it isn't the same as the one I have. And then we're going to give the URL of the buildpack that we want to use. Let's navigate back to the mars create react app buildpack that we're going to be using. And when I scroll down, I find right here the URL of the buildpack that we want.



```bash
heroku create smerth-links --buildpack https://github.com/mars/create-react-app-buildpack.git
```









By entering this into our command, we're telling Heroku use this buildpack when you assemble my app. Alright, let's press enter. Cool. So we see now that our tic tac turing instructional apphas been created on Heroku. The next thing we're going to do, is we're going to set our graph QL endpoint variable to work on Heroku as well. So, as you remember, on line 58 and line 59, I have this node environment variable that I initialized, graph QL endpoint. I can see it here. We're going to add this to our Heroku account as well.

The way we do that is we say Heroku config set, and then we just copy and paste the name of the variable and what we want it to be set to. Press enter. Good. And now our variable's been initialized. If you're having difficulty setting some of your environment variables on Heroku, here's a command that you can run from the command line. You'll say Heroku and then config and then set to set the variable you want.

Then it's the variable name, and in our case, we're going to call it graph QL endpoint. And then, you're going to say equals. And now, the name of your API endpoint. So, your API. And now, to make sure that our environment variable is configured for the right project, maybe you have multiple Heroku projects, you're going to say dash dash app. And then you're going to say the name of your Heroku project. So, mine is tic tac turing instructional, but remember, yours will have to be something different.

Since Heroku projects are globally unique, yours can't be tic tac turing instructional. Yours will be whatever you want to call yours. From now on, all we need to do to deploy is git add to add all of our files to our git repository. Commit them. We'll add the message our first Heroku commit.And then we just push them up to Heroku master. So, as you remember, when we're pushing to git hub, we pushed origin master.

This case, we're going to push to Heroku master, Heroku being the name of the remote repository, master the name of the branch. Sometimes it takes a little while for us to upload our files to Heroku and then for Heroku to assemble them into a site. So, you might sit for a second while it works. Now remember, in the exercise files, none of these individual directories have a git repository in them. So, if you want to deploy to Heroku or deploy to git from inside the exercise files, you'll need to initialize a new git repository with git in it.

Now that all that's finished, I should be able to look at my site up on Heroku by entering in the command Heroku open. This opens up a new tab in my browser, and I can see, look, here's my site. Now, there is one catch. When I do something like enter in slash profile, one of the routes that I should have available to me, I get this error. 404 not found ngenix. This is being caused by a problem with the way we're doing our routing. Essentially, the server and our routing libraryare fighting each other to interpret the URL.

Thankfully, this isn't too difficult of a bug to fix. We just need to create a file called static dot jsonin our route directory and paste in some information. I'm going to go here to my file tree. I'm going to add a new file to the root directory. We'll call this static dot json. And in here, I'm going to specify a few configuration options. Amongst them, is this one that says clean URLs false.

I'm not going to go into what all of these parts do. You're only going to have to do it once, but if you're interested about where I found this, you can go to the mars create react app buildpack git hub page, where they give you some specifications, like how to provide HTTPS, or if you want a proxy to another server. I'll save this, and now I'll go through the process of deploying this to Heroku once again, just to make sure everything works. Git add git commit dash M. Added a static dot json file.

Git push Heroku master. I'm choosing to deploy this to Heroku. Now that we're finished deploying to Heroku for the second time, let's open it up one more time and see if it works.Heroku open. And let's test out the error that we were having before. We'll enter in the URL we want to navigate to. And great, it works. If I press back, it still works. If I press forward, we're good. So now, we can deploy to Heroku just as easily as we'd push to a git repository.