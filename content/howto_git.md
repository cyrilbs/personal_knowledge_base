
# to push a new repo to github

## go to github and copy the ssh key 
```
ssh-keygen -t ed25519 -C "your_email@example.com"
cat ~/.ssh/id_ed25519.pub
```
## to push
```

git init  
git add .  
git commit -m "init quartz site"
git remote set-url origin git@github.com:yourname/your-repo.git
git push
```
