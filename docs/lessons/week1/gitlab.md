# GitLab and MKDocs Setup

While setting up GitLab, I followed <a href="https://fabacademy.org/2023/labs/charlotte/students/adam-stone/lessons/week1/gitlab/">**Adam Stone's**</a> documentation. 

## Linking Git and MKDocs

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
Unfortunately, I did not have the requirements.txt and thus I would run into an error when pushing my code to GitLab:

<pre><code>ERROR: Could not open requirements file: [Errno 2] No such file or directory: 'requirements.txt'
[notice] A new release of pip is available: 23.0.1 -> 23.3.2
[notice] To update, run: pip install --upgrade pip
real	0m2.853s
user	0m2.665s
sys	0m0.168s
Cleaning up project directory and file based variables
00:01
ERROR: Job failed: exit code 1
</code></pre>

I didn't realize that this was the error until about half an hour of debugging, as this error message was only found in the Jobs list in GitLab, which was difficult to navigate to. After I found it, however, I used the Jobs error messages as an output console and was able to easily debug any future issues. After 