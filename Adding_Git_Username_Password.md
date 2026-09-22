# How to Add Git Username, Email and Password

Add Git username and email

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

```

Run this command and then clone a repository that is private:

```bash
git config --global credential.helper store
```