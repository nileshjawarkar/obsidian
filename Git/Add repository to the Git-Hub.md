
1) Create repository on git-hub
![[Pasted image 20230827122204.png]]
2) Init local repo
``` sh
echo "# neovim-config" >> README.md
echo ".git/*" >> .gitignore
git init
git add README.md
git add .
git commit -m "Initialization for repo"
git branch -M main
git remote add origin git@github.com:nileshjawarkar/<REPO-NAME>.git
git push -u origin main
```

