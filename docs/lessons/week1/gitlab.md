# GitLab and MKDocs Setup

## Git and MKDocs

Due to having prior coding experience, I already had GitHub and MKDocs set up on my computer, which I used for documenting PreFab. As such, the biggest difficulty I encountered was configuring the .gitlab-ci.yml file to properly integrate with MKDocs.

At first, I was using an outdated file which I copied into my .gitlab-ci.yml file, which looked like this:
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

Unfortunately, I initially download my .gitlab-ci.yml file from the <a href="https://gitlab.com/pages/mkdocs">**MKDocs Gitlab repo**</a>, which meant that I was using an outdated file. Additionally, I did not download the requirements.txt file and thus I would recieve the following error when I pushed to Git:

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
I was unable to solve the error for about a half hour because I didn't know how to find the error message and I was simply modifying code snippets which I thought were problematic. However, when I checked my email, I recieved a notification that my job failed and clicked the link which brought me to the Jobs page on GitLab. The Jobs page itself was a difficult location to navigate to, and only there could I see the actual console error message output. After I was able to see the error, debugging was easy.

<center>
<img src="../../../pics/week1/jobs.jpg" alt="Jobs Menu Page" width="450"/>
</center>

<br>

I realized that I should be cloning from the <a href="https://gitlab.fabcloud.org/fibasile/fabacademy-student-template/-/tree/master?ref_type=heads">**Fab Academy Student Template GitLab repo**</a> as opposed to the <a href="https://gitlab.com/pages/mkdocs">**MKDocs GitLab repo**</a>. From there, I downloaded the requirements.txt file along with a better suited .gitlab-ci.yml file. However, I downloaded the files manually instead of cloning the repo because I only needed 2 files. Additionally, I had prior experience working with MKDocs and wanted to setup my own framework instead of using the provided one. My updated .gitlab-ci.yml file looks like this:

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

## References

I used <a href="https://fabacademy.org/2020/labs/leon/students/adrian-torres/">**Adrian Torres'**</a> and <a href="https://fabacademy.org/2023/labs/charlotte/students/adam-stone/lessons/week1/gitlab/">**Adam Stone's**</a> documentation to help me set up GitLab. 