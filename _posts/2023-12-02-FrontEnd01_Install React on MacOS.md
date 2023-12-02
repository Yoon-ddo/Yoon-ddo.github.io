---
title: FrontEnd01_Install React on MacOS
toc: true
toc_sticky: true
toc_label: "Contents of Page"
categories:
  - Study
tags:
  - Project
---

<br><br><br>

# How to Install the React on the MacOs

<br>

## 1. Download Node.js

<br>

<https://nodejs.org/en/>

<br>

I installed the latest version of the Node.js, but that was unmatched with my npm.

<br><br><br>

## 2. Download VScode

<br>

<https://code.visualstudio.com/download>

<br>

Let me show you how I solved the problem.

<br><br><br>

## 3. Command on the terminal as below

<br>

```
## 1. install the react
sudo npm install create-react-app

## 1-1. Just clean cache. It could help you to solve the unmatched problem with npm.
npm cache clean --force

## 2. check it installed perfectly. If the install is end it shows you the version. like 5.0.1
npx create-react-app -V

## 3. move and make to directory where you want to develop the program.
mkdir ~~~
cd ~~~

## 4. 
npx create-react-app .

## 6. If there are 6 files and folders are created, It is well done. 
- folders : node_modules, public, src
- files : package-lock.json, package.json, README.md

## 7. Open the VScode and turn on the "terminal" and command as below.
npm run start


```

<br><br><br>

## 4. Finish

<br>

If the screen below turns on the chrome something like that, It is the end of the install React!

<br>

![EndOfInstall](/assets/imgss/FinishReact.png)




<br><br><br>




