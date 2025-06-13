# Python environments

## General Principle 1
You must use virtual environments for python projects. It means the right environment
can be reproduced by others and future you and crucially also for deployment. It also appears to be very easy to
install packages which clash with each other and will stop things working. If 
using virtual environments this is very easy to fix and stops you from having to
re-install python.

## General Principle 2
The environment you need for development might be different to the environment you
need for deployement. For example, for development you need all the same packages as you need
for deployment PLUS a few development tools - such as pip-tools and ipykernel for
example. This is why we will create a separate list of development dependencies.

## General Principle 3
We will use conda to manage environments for development and venv to manage environments
for deployment. This is pretty normal. Venv deals with requirements.txt files,
whereas conda deals with .yml files, so we have to make both for our projects.
However, we will not make separate lists of dependencies - but just reference
our requirements.txt in the .yml.
