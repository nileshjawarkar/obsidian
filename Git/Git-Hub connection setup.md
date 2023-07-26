### Adding SSH key to git-hub account
1)  [Checking for existing SSH keys](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/checking-for-existing-ssh-keys)
2) [Generate SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
3) [Add SSH key](https://docs.github.com/en/get-started/getting-started-with-git/managing-remote-repositories#switching-remote-urls-from-https-to-ssh). 

**Some example command used **

``` sh
# Check if ssh key exist
ls -al ~/.ssh

# Create new key
cd ~/.ssh
ssh-keygen -t ed25519 -C "jawarkar.nilesh@gmail.com"
```

**Take backup of generated keys, so that it can be used in future.**

### Use existing keys

- If key already set on git-hub account, then copy keys from backup to "~/.ssh" directory
- If you lost the backup, please re-generate the keys and add it to you git-hub account

### Set email and user-name

``` sh
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```

**Now we are ready to work with our git-hub account**
