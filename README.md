vainoord@vnrd-S01:~/netology/devops-netology$ git log --oneline --decorate --graph --all
*   80ed8da (HEAD -> main, origin/main, origin/HEAD) Merge branch 'git-rebase' into main
|\  
| *   a34f480 (origin/git-rebase, git-rebase) Merge branch 'git-merge' into main
| |\  
* | \   b669969 Merge branch 'git-merge' into main
|\ \ \  
| |/ /  
|/| /   
| |/    
| * 1e36eda (origin/git-merge, git-merge) merge: use shift
| * d61d0e7 merge: @ instead *
* | 3635856 rebase: echo "====="
|/  
* d226aa0 prepare for merge and rebase
* d3aa177 (tag: v0.1, tag: v0.0, gitlab/main, bitbucket/main) First commit README.md
* cbd8833 Moved and deleted
| * 5f1de0e (origin/fix, fix) Fix file through IDE (Pycharm)
| * 76e90ee fix
|/  
* 51761ec Prepare to delete and move
* c84462c Added gitignore
* 701f4b3 Initial commit
* 3bbc4c5 (bitbucket/master) .gitignore edited online with Bitbucket
* 6b2eec1 Initial commit


My new project

# .gitignore

 Следующие файлы директории terraform не будут включены в репозиторий:
 1) все файлы в поддиректории .terraform
 2) игнорирование файлов, которые имеют в название файла расширение .tfstate
 3) файлы crash.log и начинающиеся на crash с раcширением .log
 4) файлы с расширением .tfvars, .tfvars.json
 5) файлы override.tf, override.tf.json
 6) файлы, оканчивающиеся на _override.tf и _override.tf.json
 7) файлы .terraformrc и terraform.rc

Add some change to this file and then make commit
