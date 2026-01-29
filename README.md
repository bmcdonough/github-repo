# github-repo
Documentation on proper configuration of my github repos.
## prerequisets
### github token
- login with your username, password and 2FA:  [https://github.com/settings/profile](https://github.com/settings/profile)
- bottom menu:  Developer Settings
- select:  Personal Access Tokens, then Tokens (classic)
- select:  Generate New Token (classic)
  - Note:  github-manager
  - Expiration:  No Expiration
  - Select Scopes:  **repo, read:org**
  - Generate Token

copy token to a safe place, like a password manager
### ubuntu
- starting with a minimal Ubuntu install
#### required packages
```shell
sudo apt update
sudo apt install gh git
```
#### github token
- Use the token with gh by setting the **GH_TOKEN** environment variable
```shell
echo 'export GH_TOKEN=your_token_here' >> ~/.bashrc
source ~/.bashrc
```
#### ssh config
- ~/.ssh/config
```shell
Host github.com
    HostName github.com
    User git
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/github_id_rsa

Host *
    StrictHostKeyChecking no
    UserKnownHostsFile /dev/null
    ServerAliveInterval 15
    ServerAliveCountMax 8
    GSSAPIAuthentication no
    ConnectTimeout 5
```