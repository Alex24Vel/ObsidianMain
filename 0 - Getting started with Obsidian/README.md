# Obsidian Basics
Why Obsidian? - Graph view helps stay on track despite constantly being redirected from documentation to documentation.

Tag what's relevant to you project/task and filter it easily in the Graph view. See [[ExampleGraphUsage.jpg]] to see how it looks like.

'#copypasted' tag - the info in the file was mostly or completely copy-pasted from the source and can be either ignored or replaced. Better to refer to the source material instead.

How to push changes to a repo:

1. Add remote repo where you want to push: 
```Bash
git remote add origin https://github.com/Alex24Vel/ObsidianMain.git
```   
2. Stage all files:
```Bash
git add .
```
3. Create a commit: 
```Bash
git commit -m "example message content"
```
4. Push to the remote repo ('`--set-upstream origin main`' is required only on the first push)
```Bash
git push --set-upstream origin main 

or:

git push --force
```
