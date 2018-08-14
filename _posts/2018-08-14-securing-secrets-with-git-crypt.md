---
layout: post
published: false
title: Securing secrets with git-crypt
---
Another challenge I've taken on recently is moving my content to GitHub from Bitbucket. Medium.com wrote a great article about [Github's impact on your software career](https://medium.com/@sitapati/the-impact-github-is-having-on-your-software-career-right-now-6ce536ec0b50) 

I feel like it is important that as I continue in management that I keep my technical skills relevant, and I also wanted to practice CI/CD tooling a bit more, so early this year I decided I wanted to move my content to GitHub. 

However, many of my projects had secrets in the configuration files, however, and before moving these to GitHub, I needed to clean this up. My initial solution was to use .gitignore files to remove anything that contained a secret. However, given the context of my projects (mostly container configuration), this removed a lot of the intellectual effort from my repos. 

Enter `git-crypt`. 

git-crypt uses `git smudge` and `git clean` to encrypt specified files with GPG before commit, so that your secrets are encrypted at rest in GitHub. 

The approximate setup steps are as follows: 

- Install gpg and configure keys. I followed this guide [here](https://alexcabal.com/creating-the-perfect-gpg-keypair/).
- [Build/Install git-crypt](https://github.com/AGWA/git-crypt/blob/master/INSTALL.md). To do this, I did need to install gcc. 
- For a project that previously contained secrets files, it's important to clean up your repo history: even if the files were removed via .gitignore at some point, they may still exist in the history. I used [this gist](https://gist.github.com/git-zombie/ae312e4ba24c6ad8a527788e96ad4866) to clean up these files.
- In my case, I'm [switching the remote](https://help.github.com/articles/changing-a-remote-s-url/) from Bitbucket to GitHub, so I also need to run: 
`git remote set-url origin https://github.com/USERNAME/REPOSITORY.git`

To [configure git-crypt for the new repo](https://www.schibsted.pl/blog/devops/securing-data-with-git-crypt/):

- Init the repo with `git-crypt init`
- Add your GPG key with `git-crypt add-gpg-user USER_ID`
- Configure the files to be hidden with a .gitattributes file

And finally to commit your project, make sure to run `git-crypt status -e`

