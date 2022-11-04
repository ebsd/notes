---
layout: post
title: "Mise en place du digital garden"
date: 2022-11-01 16:10:11 +0100
public: yes
---

```
$ pacman -S ruby
$ gem install bundler
$ git clone git@github.com:ebsd/digital-garden-jekyll-template.git my-digital-garden
$ cd my-digital-garden
$ bundle
$ bundle exec jekyll serve
```

Ajouter du contenu dans _notes ou _pages
puis :

```
$ git add --all
$ git commit -m 'Update content'
$ git push origin master
```

## Script python pour "publier" seulement les notes publiques

```
import sys
import os
import shutil
import git
from git import Repo

def gitpush():
  repo = git.Repo('~/my-digital-garden-repo')
  print("Adding all the working files")
  repo.git.add('--all')
  print("Commiting indexed changes ")
  repo.git.commit('-m', 'Updated content', author='who@is.it')
  origin = repo.remote(name='origin')
  print("Pushing all the committed changes")
  origin.push()


def process_directory(input_dir, output_dir):
  for file in os.listdir(input_dir):
    if file.endswith('.md'):
      curr_file_path = os.path.join(input_dir, file)

      with open(curr_file_path, "r") as f:
        content = f.read().lstrip().replace("&nbsp;", " ")
        if content.startswith("---\n") and len(content.split("---\n")) >= 3:  # yaml
            # ne publier que les notes dont "public: yes"
            if "public: " in content.split("---\n")[1]:
                if not content.split("public: ")[1].startswith("yes"):
                  print("private : " + file)
                else:
                  print("public : " + file)
                  shutil.copy2(file, output_dir)

if __name__ == '__main__':
  input_dir = os.path.join(os.getcwd(), os.path.normpath(sys.argv[1]))
  output_dir = os.path.join(os.getcwd(), os.path.normpath(sys.argv[2]))
  #print(input_dir)
  process_directory(input_dir, output_dir)
  gitpush()
```

Usage :

```
$ python3 copy-public-notes.py ~/sync/my-notes/ ~/my-digital-garden-repo/_notes/
```
