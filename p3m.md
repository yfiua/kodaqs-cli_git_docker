---
title: "Computational Environment II: Time-Stamped Repositories and Virtual Environment"
author: Chung-hong Chan
---

# Introduction

In the previous unit, we discussed how to get information about a computational environment. In this unit, we will discuss two relatively simple strategies to properly document the computational environment for automatic recreation. In the next unit, we will talk about the most comprehensive strategy thus far.

## Manual recreation

Using the commands introduced in the last unit, we can get information about the computational environment. Those information could be included alongside the shared code. However, the information can only allow us to recreate the computational environment manually, install each component one by one.

Manual recreation of a computational environment is tricky, because package installation commands usually assume one would like to install the latest version of a package. Suppose you developed your project in 2018 and you have used the most updated version of *dplyr* (0.7.8) for your project at the time. Fast forward to 2025 and you would like rerun the code in your project again. If you use `install.packages('dplyr')` to install *dplyr*, it will install the latest version (as of writing, 1.1.4) [1].

A better approach is to document your computational environment so that other researchers (and perhaps your future self) can recreate the computational environment automatically.

## Automatic recreation

A slightly better approach is to document the computational environment to enable automatic recreation. In the appendix, many strategies are listed and discussed.

In this unit, we will discussed time-stamped repositories and virtual environment. Please note that, however, these two solutions can only provide good coverage of recreating the fourth layer (software libraries). Virtual environment has a limited coverage of recreating the third layer for Python (programming language and its version)[2]. These two solutions have no coverage of the first and second layers.

## Configuration files

Automatic recreation of a computation environment is facilitated by [configuration files](https://repo2docker.readthedocs.io/en/latest/configuration/).

The simplest way to create a configuration file is to list out all packages in a computational environment.

### R

Create a file called "install.R" with a call to `install.packages()`. For example, if your code depends on *sna* and *lubridate*, put this line in "install.R"

```r
install.packages(c("sna", "lubridate"))
```

One can then recreate the computational environment either by `source("install.R")` or `Rscript install.R` (from the shell).

### Python

Create a file called "requirements.txt" and list out all required Python packages line by line. For example, if your code depends on *numpy* and *watermark*, put the follow lines in "requirements.txt"

```
numpy
watermark
```
One can then recreate the computational environment by `pip install -r requirements.txt` (from the shell).

## Time-Stamped Repositories 

As said, the issue with the above configuration files is that, install.packages() or pip installs the latest version of the requested packages from the default software repository (CRAN, pypi) as of the day one runs them.

A simple remedy to this is to use a time-stamped repository such as [Posit Public Software Repository (P3M)](https://packagemanager.posit.co/client/#/). A time-stamped repository, as the name suggested, has a time stamp and that time stamp represents the freeze date. Installing software from a time-stamped repository will install the latest version as of the freeze date, rather than the current date. As an example, if you install *dplyr* now from P3M with a freeze date of 2018-09-10, it will install the latest version as of 2018-09-10, i.e. *dplyr* 0.8.3.

P3M supports both R and Python and it is probably the easiest solution so far for the fourth layer. The idea is to generate a URL of the time stamped repository. In order to obtain a URL, go to P3M , select the appropriate repository (CRAN for R, pypi for Python), and then press the SETUP button. On the next page, select "Yes, always install packages from the date I choose" and choose the freeze date. It will generate a URL such as: `https://packagemanager.posit.co/cran/2024-10-30`

The rest of the information on that page actually tells you how to setup. But I repeat it here so that you can enhance your configuration file.

### R

In "install.R", add one more line to temporarily change the default repository to P3M, e.g.

```r
options(repos = c(CRAN = "https://packagemanager.posit.co/cran/2024-10-30"))
install.packages(c("sna", "lubridate"))
```

### Python

Similarly, temporarily change the default repository to P3M in "requirements.txt"

```
--index-url https://packagemanager.posit.co/pypi/2024-10-28/simple
--trusted-host packagemanager.posit.co
numpy
watermark
```

## Virtual environment

By using `install.packages()` or `pip` by default, these commands install software packages in the main library of your system. Therefore, you can use these packages for any project hosted on your system. In other words, all projects on your system are using the same computational environment. Let's call it the main environment. Developing your code in the main environment might be detrimental because your older projects that depend on older versions of software packages might not run after you have updated your core library. It is also difficult to keep track of which packages and their versions in a project have been used.

A solution to this is to create a virtual environment. A virtual environment is a computational environment specifically crafted for a specific project. In certain implementations (e.g. `renv`), packages installed are available exclusively for a project, i.e. not available for other projects on the same computer. The installed packages are recorded in a *lock file*. One can recreate the same virtual environment using that lock file.

For R, there is only one widely used solution (*renv*). There are many solutions on the Python front: *v(irtual)env*, *pipenv*, *poetry*, *pyenv*, *uv*, etc [3]. We will only cover *renv* and *pipenv* in this unit.

### renv

[renv](https://rstudio.github.io/renv/) helps you creating a virtual environment for a project and keeping track of the required R packages.

To understand how it works, it is better to create a toy project as a directory. After you install *renv* (`install.packages("renv"))`, then create a directory. Let's call it "hellorenv" and then start R from there.

```sh
mkdir hellorenv
cd helloenv
R
```

In the R session, use `init()` to initialize the project to use renv.

```r
renv::init()
```

The output will instruct you to restart your R session. Let's first quit the session (`q()`) and back to the shell. Use `ls` to list out the content in "hellorenv", you will see files have been added to it. Notably, `renv.lock` is the lock file of your project and all installed and required R packages will be recorded there.

Let's us relaunch R once again. You should see a message like this:

```
- Project 'hellorenv' loaded. [renv 1.0.11]
```

Now, renv is enabled in your project. Every time you use `install.packages()` to install R packages under this project, those packages will only be available to this project only. As you work on the project by creating your R scripts, *renv* will check whether the required R packages have been installed. If any of these R packages have not been installed when you launch an R session, *renv* will display a message asking you to run: `renv::status()` to check for missing dependencies. After you install those dependencies with `install.packages()`, run `renv::snapshot()` to update the lock file.

With this lock file, you can share it along side your code. Other researchers can reconstruct exactly the same computational environment (in the fourth layer) with `renv::restore()`.

### pipenv

[*pipenv*](https://pipenv.pypa.io/en/latest/) helps you creating a virtual environment for a project and keeping track of the required Python packages.

Suppose you have installed *pipenv* via `pip install pipenv`. To understand how it works, let's create a toy project called "hellopipenv".

```sh
mkdir hellopipenv
cd hellopipenv
```

Here, a way to initialize this project is to activate the pipenv virtual environment with `pipenv shell`.

```sh
pipenv shell
```

To install new Python packages, instead of using `pip install`, use `pipenv install` to install them for the virtual environment, e.g.

```sh
pipenv install numpy
```

If you run `ls` to list out all content in "hellopipenv" at this point, you will see a file called "Pipfile.lock". This file will record all the dependencies of the project. You can share this file alongside your code. Other researchers (and you) can recreate by running `pipenv sync`.

To deactivate the pipenv environment, run `exit`.

# Summary

We discussed two strategies on how to preserve the fourth layer of a computational environment: Time-stamped repositories and virtual environment.

# Footnotes

[1]: It is possible install an older version of an R package from CRAN using e.g. `remotes::install_version("dplyr", version = "0.7.8")`. However, the dependencies installed (e.g. dplyr depends on tibble) will also be the newest.

[2]: One may also use [rig](https://github.com/r-lib/rig) to install and use multiple versions of R.

[3]: There must be a lot of tools out there. In this week, I will take the "no kitchen sink" approach to introduce only one tool per each approach for each language. I cannot offer you with excess and redundancy as in the saying "everything but the kitchen sink." What's really matter for this course is to understand the general idea of an approach by examining just the one tool. The one tool introduced is usually not the best solution for all situations. You can then explore and compare many tools and have your own taste later. Sidenote: You can actually use *renv* also to create a virtual Python environment. But we will stick to the bilingual policy.

