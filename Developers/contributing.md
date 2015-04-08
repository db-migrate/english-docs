# Contributing

So happy to see you here. Any help is appreciated and so is yours!

So let us get you started!

## Guidelines

Before reading about the technical stuff, it's important to know to which
guidelines the project sticks.

## The workflow

We advise you to follow the following steps, if you plan to contribute:

1. Choose the right project you want to contribute to, either a driver or
db-migrate itself.

2. For the repo.

3. Create a new branch to work with out of the master branch.

4. Setup your environment and let the current tests run, they should pass
if not there is something wrong with your setup.

5. Be awesome!

6. Add a test for your change. Refactoring and documentation changes require no new tests. If you are adding functionality or fixing a bug, please include a test.

7. Make the test pass.

8. Pull the remote master into your master branch and rebase
your current working branch against this master.

9. Push your rebased branch and submit your pull request.

10. Make sure the travis tests in your pull request are all passing, if they
fail check why.

## Tips

 * You should never work in your master branch of your fork, you're going to
pull the changes from the master project into this branch and use your
branch to rebase against it.

 * You should pull changes from the master project regularly and rebase against
them to make your own life easier!
