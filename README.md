# ReactJs  Heroku deployment with CircleCI and CodeClimate

*How to set up continuous integration and deployment for your React app?*

*How to set up automatic deployment after any changes on master branch to Heroku after all tests pass?*

## What do you need;
* [Github repo](http://github.com/)
* [ReactJs application](https://github.com/facebook/create-react-app)
* [CircleCI account](https://circleci.com/)
* [CodeClimate account](https://codeclimate.com/)
* [Heroku account](https://www.heroku.com/) 
* [Nodejs](https://nodejs.org/en/) (Min version: 8)
* [Yarn](https://yarnpkg.com/en/) or [npm](https://www.npmjs.com/)

All of them are free. Just sign up with your github account and give permission to access your public and private repository.

### Step by step configuration
1. Create your own ReactJs app (If you have already skip to the next step)
```
$ yarn install -g create-react-app
$ create-react-app reactjs-ci-cc
$ cd reactjs-ci-cc/
$ yarn start
```
2. Configuration to setup CircleCI
    *   Create a .circleci folder under project's root folder and create a config.yml file in .circleci folder.
    *   Add following;
    ```
    version: 2
    jobs:
    build:
        docker:
        - image: circleci/node:8

        working_directory: ~/repo
        
        steps:
        - checkout
        - restore_cache: # special step to restore the dependency cache
            key: dependency-cache-{{ checksum "package.json" }}
        - run:
            name: Setup Dependencies
            command: yarn install
        - run:
            name: Setup Code Climate test-reporter
            command: |
                curl -L https://codeclimate.com/downloads/test-reporter/test-reporter-latest-linux-amd64 > ./cc-test-reporter
                chmod +x ./cc-test-reporter
        - save_cache: # special step to save the dependency cache
            key: dependency-cache-{{ checksum "package.json" }}
            paths:
                - ./node_modules
        - run: # run tests
            name: Run Test and Coverage
            command: |
                ./cc-test-reporter before-build
                yarn test -- --coverage
                ./cc-test-reporter after-build --exit-code $?
    ```
3. Setup CodeClimate Repo
    * Go to [CodeClimate](https://codeclimate.com/) page
    * Add your repo from your github page
    * Get ```Test Reporter ID``` from ```Settings > Test Coverage``` of your project on CodeClimate page.
    * Keep it in somewhere to add into CircleCI configuration step.
4. Create a repo in [Github](http://github.com/)
```
$ git init
$ git remote add origin git@github.com:username/new-repo-here
$ git add .
$ git commit -m “first commit”
$ git push -u origin master
```
5. Build and Test the Project
    * Go to [CircleCI](https://circleci.com/) webpage
    * Add your project from your github page
    * Navigate to ```Project > Settings > Environment variable``` and add ```CC_TEST_REPORTER_ID``` with the copied ```Test Reporter ID```
    * Go back to build page
    * Start to build for your new project.
6. Heroku Deployment Step
    * To set Heroku buildpack, use following command
    ```
    $ heroku create --buildpack https://github.com/mars/create-react-app-buildpack.git -a reactjs-ci-cc'
    ```
    * We pushed the latest master branch to heroku with git push heroku master
    * Go to your project's detail page from your heroku dashboard
    * Select ```Deploy``` tab
    * Choose ```Github``` option from ```Deployment Method```
    * Enable automatic deployment and check ```Wait for CI to pass before deploy```
7. Congratulations you are done.

### To test full cycle
1. Do some code change in your local repo
2. Commit and push your changes to the github repo
3. Go to CircleCI dashboard and choose your project
4. Check build steps, be sure there is no any failure.
5. Go to heroku dashboard and choose your project. Navigate to the 'Activity' tab
6. if you are seeing 'Build succeeded' under 'Activity Feed' then;
7. Click 'Open App' button on the top right corner.
8. Check your changes on your application heroku page.


To setup and understand all these configurations I would like to say thank [Zac Kwan](https://medium.freecodecamp.org/@Zaccc123) for his awesome explanation on his [medium page](https://medium.freecodecamp.org/how-to-set-up-continuous-integration-and-deployment-for-your-react-app-d09ae4525250).
