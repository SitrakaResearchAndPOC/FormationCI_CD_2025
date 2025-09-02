# First example 
* test 1
```
name: My First Workflow

on: push

jobs:
  first_job:
    runs-on: ubuntu-latest
    steps:
      - name: Welcome message
        run: echo "My first Github Actions Job"

      - name: List files
        run: ls

      - name: Read file
        run: cat README.md
```
* test2 
```
name: My First Workflow

on: push

jobs:
  first_job:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Repo
      uses: actions/checkout@v4

    - name: List and Read file
      run: |
        echo "My first Github Actions Job"
        ls -ltra
        cat README.md
```
*test 3 
```
name: My First Workflow v2

on: push

jobs:
  first_job_actions1:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Repo
      uses: actions/checkout@v4

    - name: List and Read file
      run: |
        echo "My first Github Actions Job"
        ls -ltra
        cat README.md
    - name: Generate ASCII Artwork
      run: |
        sudo apt update
        sudo apt install cowsay
        cowsay -f dragon "Run for cover, I am a DRAGON...RAWR" >> dragon.txt
```
* test 4 :
```
name: Generate ASCII Artwork

on: push

jobs:
  acti_job:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Repo
      uses: actions/checkout@v4

    - name: Install Cowsay Program
      run: sudo apt-get install cowsay -y

    - name: Execute Cowsay Command
      run: cowsay -f dragon "Run for cover, I am a DRAGON...RAWR" >> dragon.txt

    - name: Test File Exists
      run: grep -i "dragon" dragon.txt

    - name: Read File
      run: cat dragon.txt

    - name: Checkout Repo Files
      run: ls
```
* test 5 : shell
ascii-script.sh
```
#!/bin/sh
sudo apt-get install cowsay -y
cowsay -f dragon "Run for cover, I am a DRAGON....RAWR" >> dragon.txt
grep -i "dragon" dragon.txt
cat dragon.txt
```
```
name: Generate ASCII Artwork

on: push

jobs:
  ascii_job:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Repo
      uses: actions/checkout@v4

    - name: List Repo Files
      run: ls -ltra

    - name: Executing Shell Script
      run: |
        chmod +x ascii-script.sh
        ./ascii-script.sh
```

* test 6 : 
```
name: Generate ASCII Artwork

on: push

jobs:
  build_job_1:
    runs-on: ubuntu-latest
    steps:
    - name: Install Cowsay Program
      run: sudo apt-get install cowsay -y

    - name: Execute Cowsay CMD
      run: cowsay -f dragon "Run for cover, I am a DRAGON....RAWR" >> dragon.txt

    - name: Sleep for 30 seconds
      run: sleep 30

  test_job_2:
    runs-on: ubuntu-latest
    steps:
    - name: Sleep for 10 seconds
      run: sleep 10

    - name: Test File Exists
      run: grep -i "dragon" dragon.txt

  deploy_job_3:
    runs-on: ubuntu-latest
    steps:
    - name: Read File
      run: cat dragon.txt

    - name: Deploy
      run: echo Deploying ...
```
* test 7
```
name: Generate ASCII Artwork

on: push

jobs:
  build_job_1:
    runs-on: ubuntu-latest
    steps:
    - name: Install Cowsay Program
      run: sudo apt-get install cowsay -y

    - name: Execute Cowsay CMD
      run: cowsay -f dragon "Run for cover, I am a DRAGON....RAWR" >> dragon.txt

    - name: Sleep for 30 seconds
      run: sleep 30

  test_job_2:
    runs-on: ubuntu-latest
    needs: build_job_1
    steps:
    - name: Sleep for 10 seconds
      run: sleep 10

    - name: Test File Exists
      run: grep -i "dragon" dragon.txt

  deploy_job_3:
    runs-on: ubuntu-latest
    needs: test_job_2
    steps:
    - name: Read File
      run: cat dragon.txt

    - name: Deploy
      run: echo Deploying ...
```

* test 8 
```
name: Generate ASCII Artwork

on: push

jobs:
  build_job_1:
    runs-on: ubuntu-latest
    steps:
    - name: Install Cowsay Program
      run: sudo apt-get install cowsay -y

    - name: Execute Cowsay CMD
      run: cowsay -f dragon "Run for cover, I am a DRAGON....RAWR" >> dragon.txt

    - name: Sleep for 30 seconds
      run: sleep 30

    - name: Upload dragon.txt artifact
      uses: actions/upload-artifact@v4
      with:
        name: dragon-file
        path: dragon.txt

  test_job_2:
    runs-on: ubuntu-latest
    needs: build_job_1
    steps:
    - name: Download dragon.txt artifact
      uses: actions/download-artifact@v4
      with:
        name: dragon-file

    - name: Sleep for 10 seconds
      run: sleep 10

    - name: Test File Exists
      run: grep -i "dragon" dragon.txt

    - name: Upload artifact for deployment
      uses: actions/upload-artifact@v4
      with:
        name: tested-dragon-file
        path: dragon.txt

  deploy_job_3:
    runs-on: ubuntu-latest
    needs: test_job_2
    steps:
    - name: Download tested file
      uses: actions/download-artifact@v4
      with:
        name: tested-dragon-file

    - name: Read File
      run: cat dragon.txt

    - name: Deploy
      run: echo Deploying ...
```
