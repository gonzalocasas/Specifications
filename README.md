What is this repository ?
=========================
This repository drafts specifications of The Things Network architecture. **The content of this
repository is still under review** and shouldn't be considered as the official specifications for
the moment.
Once done, the whole repository will be transfered to The Things Network's organization. At
this time, it will be fully considered as a document you can rely on to contribute to the
architecture. 

How to contribute ?
========================
Any contributor is welcomed and could give a hand in several manners:

- Looking for and correcting typos
- Add relevant and non redundant information
- Complete a part marked as //TODO

**Regardless of what you'll attempt, make sure to check [the issues list on
GitHub][github_issues] before doing anything**. Thus, if nobody is already working on something
related, open an issue with a clear and concise title and briefly explain your incoming changes
in the description. Then, fork the project, do your work, and create a Pull Request on the
original repository. 

To sum up:

- Check [GitHub issues][github_issues]
- Open a new issue if no one exist
- Fork the repository
- Create a Pull Request once you're done

How to setup tools ?
======================

First of all make sure to have python installed (`^2.7.x`). 

```sh
python --version
# Python 2.7.6
```

Then, install `pip`. 

```sh
# Ubuntu / Debian
sudo apt-get install python-pip

or 

# Fedora
sudo yum install python-pip

or 

#Mac OSX
brew install python
```

And finally, install `mkdocs`

```sh
sudo pip install mkdocs
```

Please, refer to [mkdocs documentation](http://mkdocs.org) for any issue. You can then work
using `mkdocs serve` to create a local server hosted by default on port `8000`. Then, to
release on gh-pages, just use `mkdocs gh-deploy`. 

**Make sure you're up-to-date with the remote before pushing**. In case of rejection, do not
merge the remote into your local repository but do a rebase of your master branch instead. 

*<p align="center">Thanks a lot !</p>*

[github_issues]: https://github.com/KtorZ/Specifications/issues
