# GitLab and MkDocs 

## Git and Python

Due to prior coding experience, I already had both <a href="https://git-scm.com/downloads">**Git**</a> and <a href="https://www.python.org/downloads/">**Python**</a> installed on my computer. I had installed <a href="https://www.mkdocs.org/">**MkDocs**</a> a few months prior for documenting some of my coding projects.

However, when I was helping some of my friends, I noticed a few common errors:

- <span style="color:darkred">The term 'python' is not recognized as the name of a cmdlet, function, script file, or operable program.</span>
    - Add the Python installation directory (where Python is located on your local system) to your System Environment Variables. 
        - To access the System Environment Variables menu, open the Run menu (Windows + R) and type in "sysdm.cpl", or go to the "View Advanced System Settings" menu. Then, click Advanced on the top navigation bar and click Environment Variables. Add the Python installation location to the PATH variable (user or system) by clicking Path -> Edit -> New.
- <span style="color:darkred">The term 'pip' is not recognized as the name of a cmdlet, function, script file, or operable program.</span>
    - Run the "<span style="color:blue">python -m ensurepip</span>"
 command. This will download pip automatically if you do not already have it.
    - Use the pip3 command instead of pip (e.g. "<span style="color:blue">pip3 install mkdocs</span>" instead of "<span style="color:blue">pip install mkdocs</span>"). 
- <span style="color:darkred">The term 'mkdocs' is not recognized as the name of a cmdlet, function, script file, or operable program.</span>
    - Add the python Scripts folder to your System Environment Variables. After running "<span style="color:blue">pip install mkdocs</span>", the mkdocs.exe file should have been added to your Scripts folder. The scripts folder can be found in the same place Python is located. For example, I installed Python under "C:\myPython", so my MkDocs was at "C:\myPython\Scripts\mkdocs.exe" and I would add "C:\myPython\Scripts" to my PATH system environment variable. The steps to edit System Environment Variables are above.
- <span style="color:darkred">fatal: not a git repository (or any of the parent directories): .git</span>
    - You could be in the wrong directory, in which case run "<span style="color:blue">cd your-git-repo-path</span>" and re-run the command. For example, I would run "<span style="color:blue">cd C:\fab\richard-shan</span>".
    - If the repository doesn't exist or you haven't created it yet, run "<span style="color:blue">git clone git-repo-url</span>".

## SSH Key Setup

To generate my SSH key, I followed the following steps. I used Windows Powershell as my terminal of choice, though others such as CMD and Git Bash would have achieved the same function.

- Open a terminal instance
- Run <span style="color:blue">git config -–global user.email “26shanr@charlottelatin.org”</span>.
- Run <span style="color:blue">ssh-keygen -t rsa -C “26shanr@charlottelatin.org”</span>.
- Specify a file to save the key into (click Return/Enter for none)
- Specify a password (click Return/Enter for none)
- Copy the generated key for storage. The key starts with SHA256 and ends with the closing quotation mark after your email address.


I then added the SSH key to my GitLabs account to allow pushing.
<br><center>
<img src="../../../pics/week1/SSHkey.jpg" alt="Adding SSH Key to GitLabs" width="700"/>
</center>
<br> <br>

To view the key if you ever lose it, you can run "<span style="color:blue">cat ~/.ssh/id_rsa.pub</span>".

## Environment Setup

I navigated to the file directory that I was going to use, then cloned my repository from GitLabs.

<pre><code class="language-none">PS C:\fab> git clone git@gitlab.fabcloud.org:academany/fabacademy/2024/labs/charlotte/students/richard-shan.git
Cloning into 'richard-shan'...
remote: Enumerating objects: 176, done.
remote: Counting objects: 100% (100/100), done.
remote: Compressing objects: 100% (83/83), done.
Receiving objects:  63% (111/176), 1.39 MiB | 2.76 MiB/seused 76
Receiving objects: 100% (176/176), 3.09 MiB | 3.39 MiB/s, done.
Resolving deltas: 100% (49/49), done.
PS C:\fab>
</code></pre>

<br>
Cloning from Github created a new folder which I subsequently navigated into.

<center>
<img src="../../../pics/week1/gitFolder.jpg" alt="Git richard-shan folder" width="450"/>
</center>
<br>

Having installed MkDocs via pip, I ran "<span style="color:blue">mkdocs new richard-shan</span>" in the top-level fab folder to initialize a new MkDocs project in my git directory. I then ran "<span style="color:blue">mkdocs serve</span>" to locally host my website for testing and viewing.

<pre><code class="language-none">PS C:\fab> mkdocs new richard-shan
INFO    -  Creating project directory: richard-shan
INFO    -  Writing config file: richard-shan\mkdocs.yml
INFO    -  Writing initial docs: richard-shan\docs\index.md
PS C:\fab> cd .\richard-shan\
PS C:\fab\richard-shan> mkdocs serve
INFO    -  Building documentation...
INFO    -  Cleaning site directory
INFO    -  Documentation built in 0.25 seconds
INFO    -  [19:57:25] Watching paths for changes: 'docs', 'mkdocs.yml'
INFO    -  [19:57:25] Serving on http://127.0.0.1:8000/
</code></pre>

<br>
The groundwork for my website was now complete, and I could view the website on my localhost. After pushing to Git and changing the default markdown files, the site was ready!

## Git and MkDocs Integration

The biggest difficulty I encountered was configuring the gitlab-ci.yml file to properly integrate with MkDocs. At first, I was using an outdated file which I copied into my gitlab-ci.yml file, which looked like this:
<pre><code class="language-yml">	
@Override
	image: python:3.8-buster

before_script:
  - pip install -r requirements.txt

test:
  stage: test
  script:
  - mkdocs build --strict --verbose --site-dir test
  artifacts:
    paths:
    - test
  rules:
    - if: $CI_COMMIT_REF_NAME != $CI_DEFAULT_BRANCH

pages:
  stage: deploy
  script:
  - mkdocs build --strict --verbose
  artifacts:
    paths:
    - public
  rules:
    - if: $CI_COMMIT_REF_NAME == $CI_DEFAULT_BRANCH
</code></pre>
<br> 

Unfortunately, I initially download my gitlab-ci.yml file from the <a href="https://gitlab.com/pages/mkdocs">**MkDocs Gitlab repo**</a>, which meant that I was using an outdated file. Additionally, I did not download the requirements.txt file and thus I would recieve the following error when I pushed to Git:

<pre><code class="language-none">ERROR: Could not open requirements file: [Errno 2] No such file or directory: 'requirements.txt'
[notice] A new release of pip is available: 23.0.1 -> 23.3.2
[notice] To update, run: pip install --upgrade pip
real	0m2.853s
user	0m2.665s
sys	0m0.168s
Cleaning up project directory and file based variables
00:01
ERROR: Job failed: exit code 1
</code></pre>
<br>
I was unable to solve the error for about a half hour because I didn't know how to find the error message and I was stuck modifying code snippets which I thought were problematic. However, when I checked my email, I recieved a notification that my job failed and clicked the link which brought me to the Jobs page on GitLab. The Jobs page itself was a difficult location to navigate to, and only there could I see the actual console error message output. However, after I was able to see the error, debugging was easy.

<center>
<img src="../../../pics/week1/jobs.jpg" alt="Jobs Menu Page" width="450"/>
</center>

<br>

I realized that I should be cloning from the <a href="https://gitlab.fabcloud.org/fibasile/fabacademy-student-template/-/tree/master?ref_type=heads">**Fab Academy Student Template GitLab repo**</a> as opposed to the <a href="https://gitlab.com/pages/mkdocs">**MkDocs GitLab repo**</a>. From there, I downloaded the requirements.txt file along with a better suited .gitlab-ci.yml file. However, I downloaded the files manually instead of cloning the repo because I only needed 2 files. Additionally, I had prior experience working with MkDocs and wanted to setup my own framework instead of using the provided one. My updated .gitlab-ci.yml file looks like this:

<pre><code class="language-yml">image: python:3.9-slim

before_script:
  - time apt update && apt-get install -y git
  - time pip install -r requirements.txt

test:
  stage: test
  script:
  - time mkdocs build --site-dir test
  artifacts:
    paths:
    - test
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
      when: never

pages:
  stage: deploy
  variables:
    ENABLED_GIT_REVISION_DATE: "true"
    GIT_DEPTH: 1000
  script:
  - time mkdocs build --site-dir public
  artifacts:
    paths:
    - public
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
</code></pre>

After updating my gitlab-ci file, the mkdocs.yml file was recognized and took precedence. My site changed to use the markdown files as opposed to the html files. The commits and pushes all succeeded and I was able to start working on the site with MkDocs.

<center>
<img src="../../../pics/week1/passed.jpg" alt="Successful git push" width="450"/>
</center>
<br>

## MkDocs File Organization

I decided not to use the Fab Student Template for setting up the week folders as I wanted to build my own directory system for documentation. The following image shows the file hierarchy that I built, starting from the root directory of the site at the top. Each page of this website is stored in a markdown (.md) file, which are stored under their respective week folders under the general lessons folder. Every image/video on this website is stored under its respective week folder under the pics folder. MkDocs's default starting directory is the docs folder, and as such, all of my documentation is stored in its subfolders.

<center>
<img src="../../../pics/week1/mkdocs file structure.jpg" alt="MkDocs File Directory" width="450"/>
</center>

<br>

## References

Huge thank you to <a href="https://fabacademy.org/2023/labs/charlotte/students/stuart-christhilf/">**Stuart Christhilf**</a> for teaching me MkDocs during our Data Structures class last semester! <a href="https://fabacademy.org/2020/labs/leon/students/adrian-torres/">**Adrian Torres'**</a> and <a href="https://fabacademy.org/2023/labs/charlotte/students/adam-stone/lessons/week1/gitlab/">**Adam Stone's**</a> documentations were also valuable resources for setting up GitLab. 