## Adding SSH key to github account
- [Checking for existing SSH keys](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/checking-for-existing-ssh-keys)
- [Generate SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
- [Add SSH key](https://docs.github.com/en/get-started/getting-started-with-git/managing-remote-repositories#switching-remote-urls-from-https-to-ssh)

## Some example command used -

``` sh
ls -al ~/.ssh
cd ~/.ssh
ssh-keygen -t ed25519 -C "jawarkar.nilesh@gmail.com"
```

## Take backup of generated keys, so that it can be used in future.

