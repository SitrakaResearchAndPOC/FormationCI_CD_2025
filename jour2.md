Generate key ssh before : 
```
ssh-keygen -t rsa -b 4096 -C "ton-email@example.com"
```

My TOKEN : 
Regener votre token :
```
https://github.com/settings/personal-access-tokens
```
```
SitrakaResearchAndPOC
```
```
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
```

## Create github
…or create a new repository on the command line
```
echo "# FormationCI_CD_2025_gitactions_python" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/SitrakaResearchAndPOC/FormationCI_CD_2025_gitactions_python.git
git push -u origin main
```
…or push an existing repository from the command line
```
git remote add origin https://github.com/SitrakaResearchAndPOC/FormationCI_CD_2025_gitactions_python.git
git branch -M main
git push -u origin main
```
## Git action java project 1
```
git clone https://github.com/nanuchi/my-project
```
```
cd my-project
```
```
rm -rf .github/workflows/ci.yml 
```
```
git init
```
```
git remote add origin https://github.com/SitrakaResearchAndPOC/ga-java-1.git
```
```
git branch -M main
```
```
git push -u origin main
```
Pipeline 1 : 
# This workflow will build a Java project with Gradle
# For more information see: https://help.github.com/actions/language-and-framework-guides/building-and-testing-java-with-gradle
```
name: Java CI with Gradle

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-java:

    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2

    - name: Set up JDK 1.8
      uses: actions/setup-java@v1
      with:
        java-version: 1.8

    - name: Grant execute permission for gradlew
      run: chmod +x gradlew

    - name: Build with Gradle
      run: ./gradlew build
```
Pipeline 2 : 
```

```


## Git simple python add actions 

```
git clone https://github.com/iam-veeramalla/GitHub-Actions-Zero-to-Hero.git
```
```
cd GitHub-Actions-Zero-to-Hero/
```
```
echo "# FormationCI_CD_2025_gitactions_python" >> README.md
```
 
