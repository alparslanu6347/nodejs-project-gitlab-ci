# nodejs-project-gitlab-ci

  ***The application's files and folders were not added to the repository because they take up too much space. However, by following the steps in the README.md file, you can generate the necessary project files and folders yourself.*** 
  ***Only the `docusaurus.config.ts` file from the application files was added to this repository because it requires two modifications. It was included in the repository as an example.***

## Part-1 : 

- Go to gitlab

- Create gitlab project/repository
  Click `+` --> Select `New project/repository` -->> Click `Create blank project`
  `Project name` : `nodejs-project`
  `Project URL`  : `https://gitlab.com/arrowlevent/nodejs-project`
  `Visibility level` : `Public`
  `Project Configurations` : Put check mark on the `Initialize repository with a README`
  Click `Create project`

## Part-2 : Launch an ec2-instance on AWS and install `nodejs`

- Go to AWS Management Console, we need a `nodejs-server` and we need to install agent into that server. (İSTERSENİZ LOKALDE DE nodejs KURULUMU YAPABİLİRSİNİZ, Ben burada AWS Amazon Linux 2023 tercih ettim.)

- Launch an AWS EC2 instance of Amazon Linux 2023 AMI `t3a.medium` with security group allowing `SSH:22`, `HTTP:80` and `TCP 8080` ports.

- Make a remote-ssh VSC connection to your instance

```bash
ssh -i .ssh/call-training.pem ec2-user@ec2-3-133-106-98.us-east-2.compute.amazonaws.com
```

- You should write these commands into `.bashrc` file, first go to bottom line and hit enter 2 times then write

```bash (.bashrc)
export PS1="\[\e[1;34m\]\u\[\e[33m\]@\h# \W:\[\e[34m\]\\$\[\e[m\] "  # yellow-blue-yellow
sudo hostnamectl set-hostname "nodejs-server"
```

```bash (pwd : home/ec2-user)
bash
bash

# Update the installed packages and package cache on your instance.
sudo dnf update -y

# install git
sudo dnf install git -y
git -v      # git version 2.40.1

# https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-up-node-on-ec2-instance.html
# Install node version manager (nvm) by typing the following at the command line.
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash # .nwm klasörü geldi

# Activate nvm by typing the following at the command line.
. ~/.nvm/nvm.sh

# Use nvm to install the latest LTS version of Node.js by typing the following at the command line.
nvm install --lts

# Test that Node.js is installed and running correctly by typing the following at the command line.
# This displays the following message that shows the version of Node.js that is running. Running Node.js VERSION

node -e "console.log('Running Node.js ' + process.version)"   # Running Node.js v20.10.0
```

```bash (pwd : home/ec2-user)
git clone https://*****TOKEN*****@gitlab.com/arrowlevent/nodejs-project.git

cd nodejs-project/

npm init docusaurus # çıkan sorulara yes ve/veya enter

# The initialization wizard sets up the site in website/, but the site should be in the root of the project. Move the files up to the root and delete the old directory:
mv website/* .
rm -r website
```

## Part-3 : Update the Docusaurus configuration file with the details of your GitLab project. In `docusaurus.config.ts`:

   - Set url: to a path with this format: https://<my-username>.gitlab.io/  
   - `11. satır` `url: 'https://your-docusaurus-site.example.com'`     --> `url: 'https://arrowlevent.gitlab.io/'`

   - Set baseUrl: to your project name, like /my-pipeline-tutorial-project/ 
   - `14. satır` `baseUrl: '/'`                                        --> `baseUrl: '/nodejs-project/'`

- Push Application to GitLab

```bash (pwd : nodejs-project)
git add .
git config --global user.email "alparslanu6347@gmail.com"
git config --global user.name "arrowlevent"
git commit -m "Add simple generated Docusaurus site"
git push origin     # şifre soracak -> TOKEN cpoy-paste enter
```

## Part-4 : Prepare `.gitlab-ci.yml` in the `root` of the project with this configuration

- Go to gitlab
  - Click and get into your project/repository `nodejs-project` --> Click `+` Select ``New file` from the list
    `filename` : `.gitlab-ci.yml`
    copy-paste into this file

`image`: Tell the runner which Docker container to use to run the job in. The runner:
         Downloads the container image and starts it.
         Clones your GitLab project into the running container.
         Runs the script commands, one at a time.

`artifacts`: Jobs are self-contained and do not share resources with each other. If you want files generated in one job to be used in another job, you must save them as artifacts first. Then later jobs can retrieve the artifacts and use the generated files.

- `build-job`: 
   - Use `image` to configure the job to run with the latest `node` image. Docusaurus is a` Node.js` project and the `node` image has the needed `npm` commands built in.
   - Run `npm install` to install Docusaurus into the running `node` container, then run `npm run build` to build the site.
   - `Docusaurus` saves the built site in `build/`, so save these files with `artifacts`


```yaml (.gitlab-ci.yml)
build-job:
  image: node
  script:
    - npm install
    - npm run build
  artifacts:
    paths:
      - "build/"
```
- Click `Commit changes`

- Pipeline aşamalarını/joblarını buradan gözlemleyebilirsin`Build` -- `Pipelines`

- Use the pipeline editor to commit this pipeline configuration to the default branch, and check the job log. You can:
    See the npm commands run and build the site.
    Verify that the artifacts are saved at the end.
    Browse the contents of the artifacts file by selecting Browse to the right of the job log after the job completes.


## Part-5 : Add a job to deploy the site

- `GitLab Pages`: To host your static site, you will use GitLab Pages. [https://docs.gitlab.com/ee/user/project/pages/index.html]

- In this step:
    Add a job that fetches the built site and deploys it. When using `GitLab Pages`, the job is always named pages. The artifacts from the `build-job` are fetched automatically and extracted into the job. Pages looks for the site in the `public/` directory though, so add a `script` command to move the site to that directory.
    Add a `stages` section, and define the stages for each job. `build-job` runs first in the `build` stage, and `pages` runs after in the `deploy` stage.

- Modify .gitlab-ci.yml

```yaml (.gitlab-ci.yml)
stages:          # List of stages for jobs and their order of execution
  - build
  - deploy

build-job:
  stage: build   # Set this job to run in the `build` stage
  image: node
  script:
    - npm install
    - npm run build
  artifacts:
    paths:
      - "build/"

pages:
  stage: deploy  # Set this new job to run in the `deploy` stage
  script:
    - mv build/ public/
  artifacts:
    paths:
      - "public/"
```
- Click `Commit changes`

- Pipeline aşamalarını/joblarını buradan gözlemleyebilirsin`Build` -- `Pipelines`

- After the pages job completes a pages-deploy job appears, which is the GitLab process that deploys the Pages site. When that job completes, you can visit your new Docusaurus site. The Pages documentation explains the URL formatting, which should be similar to https://<my-username>.gitlab.io/<my-pipeline-tutorial-project>/.

- Static web sitesini görmek için : `https://arrowlevent.gitlab.io/nodejs-project/`


## Part-6 : Add test jobs

- Now that the site builds and deploys as expected, you can add tests and `linting`. For example, a Ruby project might run RSpec test jobs. Docusaurus is a static site that uses Markdown and generated HTML, so this tutorial adds jobs to test the Markdown and HTML.
  - `linting` = Source kodunuzdaki hataları, uyumsuzlukları veya belirli kurallara uymayan unsurları tespit etmek için kullanılan bir süreci ifade eder. Linting, genellikle yazım hataları, kod stili ihlalleri ve potansiyel hata kaynakları gibi konuları tespit ederek kod kalitesini artırmaya yardımcı olur. Bu süreç, yazılım geliştirme sürecinde standartlar ve belirli kuralların takip edilmesini sağlamak amacıyla kullanılır. Linting araçları, belirli bir dil veya teknoloji için kodunuzu analiz eder ve belirli stil kılavuzlarına uymayan kodu belirler. 

- This Part-6 introduces:
   - `allow_failure`: Jobs that fail intermittently, or are expected to fail, can slow down productivity or be difficult to troubleshoot. Use allow_failure to let jobs fail without halting pipeline execution. [https://docs.gitlab.com/ee/ci/yaml/?query=allow_failure#allow_failure]
   - `dependencies`: Use `dependencies` to control artifact downloads in individual jobs by listing which jobs to fetch artifacts from. [https://docs.gitlab.com/ee/ci/yaml/?query=dependencies#dependencies]

- In this Part-6:
  - Add a new `test` stage that runs between `build` and `deploy`. These three stages are the default stages when `stages` is undefined in the configuration.

  - Add a `lint-markdown` job to run `markdownlint` and check the Markdown in your project. markdownlint is a static analysis tool that checks that your Markdown files follow formatting standards.
    - The sample Markdown files Docusaurus generates are in `blog/` and `docs/`. These directories in the root of project/repository.

    - This tool scans the original Markdown files only, and does not need the generated HTML saved in the `build-job` artifacts. Speed up the job with `dependencies: []` so that it fetches no artifacts.

    - A few of the sample Markdown files violate default markdownlint rules, so add `allow_failure: true` to let the pipeline continue despite the rule violations.

  - Add a `test-htm`l job to run `HTMLHint` and check the generated HTML. HTMLHint is a static analysis tool that scans generated HTML for known issues.

  - Both `test-html` and `pages` need the generated HTML found in the `build-job` artifacts. Jobs fetch artifacts from all jobs in earlier stages by default, but add `dependencies:` to make sure the jobs don’t accidentally download other artifacts after future pipeline changes.

- Modify .gitlab-ci.yml

```yaml (.gitlab-ci.yml)
stages:
  - build
  - test               # Add a `test` stage for the test jobs
  - deploy

build-job:
  stage: build
  image: node
  script:
    - npm install
    - npm run build
  artifacts:
    paths:
      - "build/"

lint-markdown:
  stage: test
  image: node
  dependencies: []     # Don't fetch any artifacts
  script:
    - npm install markdownlint-cli2 --global           # Install markdownlint into the container
    - markdownlint-cli2 -v                             # Verify the version, useful for troubleshooting
    - markdownlint-cli2 "blog/**/*.md" "docs/**/*.md"  # Lint all markdown files in blog/ and docs/
  allow_failure: true  # This job fails right now, but don't let it stop the pipeline.

test-html:
  stage: test
  image: node
  dependencies:
    - build-job        # Only fetch artifacts from `build-job`
  script:
    - npm install --save-dev htmlhint                  # Install HTMLHint into the container
    - npx htmlhint --version                           # Verify the version, useful for troubleshooting
    - npx htmlhint build/                              # Lint all markdown files in blog/ and docs/

pages:
  stage: deploy
  dependencies:
    - build-job        # Only fetch artifacts from `build-job`
  script:
    - mv build/ public/
  artifacts:
    paths:
      - "public/"
```
- Click `Commit changes`

- Pipeline aşamalarını/joblarını buradan gözlemleyebilirsin`Build` -- `Pipelines`

- The `lint-markdown` job in the `test stage` fails because the sample Markdown violates the default markdownlint rules, but is allowed to fail.
  *You can Add a markdownlint configuration file to limit which rule violations to alert on.*
  *You can also make changes to the Markdown file content and see the changes on the site after the next deployment.*
  BU `test stage`İÇİNDEKİ `lint-markdown` job FAILED OLDU AMA PIPELINE ÇALIŞTI.
  BU JOB'U ÇALIŞTIRMAK İÇİN `blog` ve `docs` klasörlerinin içindeki tüm `.md` uzantılı `Markdown` dosyalarını modifiye etmelisiniz ve bu job içindeki `allow_failure: true` satırını ya komple kaldırmalısın ya da `allow_failure: false` olarak değiştirmelisin. BEN AYNEN BIRAKTIM.

- Static web sitesini görmek için : `https://arrowlevent.gitlab.io/nodejs-project/`

## Part-7 : Start using merge request pipelines


- This Part-7 introduces:

  - `rules:` Add rules to each job to configure in which pipelines they run. You can configure jobs to run in `merge request pipelines`, `scheduled pipelines`, or other `specific situations`. Rules are evaluated from top to bottom, and if a rule matches, the job is added to the pipeline.[https://docs.gitlab.com/ee/ci/yaml/index.html#rules]

  - `CI/CD variables`: use these environment variables to configure job behavior in the configuration file and in script commands. `Predefined CI/CD variables` are variables that you do not need to manually define. They are automatically injected into pipelines so you can use them to configure your pipeline. Variables are usually formatted as `$VARIABLE_NAME`. and predefined variables are usually prefixed with `$CI_`. [https://docs.gitlab.com/ee/ci/variables/index.html] [https://docs.gitlab.com/ee/ci/variables/predefined_variables.html]

- In this Part-7:

  - Create a ***new feature branch*** and make the changes in the branch instead of the default branch.
    - Click and get into your project/repository `nodejs-project` --> Click `+` Select ``New branch` from the list
    `Branchname` : `feature1`   `Create from` : `main`  -->> Create

  - Add `rules` to each job:
    - The site should only deploy for changes to the default branch.
    - The other jobs should run for all changes in merge requests or the default branch.
  
  - With this pipeline configuration, you can work from a feature branch without running any jobs, which saves resources. When you are ready to validate your changes, create a merge request and a pipeline runs with the jobs configured to run in merge requests.

  - When your merge request is accepted and the changes merge to the default branch, a new pipeline runs which also contains the `pages` deployment job. The site deploys if no jobs fail.


- Modify .gitlab-ci.yml

```yaml (.gitlab-ci.yml)
stages:
  - build
  - test
  - deploy

build-job:
  stage: build
  image: node
  script:
    - npm install
    - npm run build
  artifacts:
    paths:
      - "build/"
  rules:
    - if: $CI_PIPELINE_SOURCE == 'merge_request_event'  # Run for all changes to a merge request's source branch
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH       # Run for all changes to the default branch

lint-markdown:
  stage: test
  image: node
  dependencies: []
  script:
    - npm install markdownlint-cli2 --global
    - markdownlint-cli2 -v
    - markdownlint-cli2 "blog/**/*.md" "docs/**/*.md"
  allow_failure: true
  rules:
    - if: $CI_PIPELINE_SOURCE == 'merge_request_event'  # Run for all changes to a merge request's source branch
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH       # Run for all changes to the default branch

test-html:
  stage: test
  image: node
  dependencies:
    - build-job
  script:
    - npm install --save-dev htmlhint
    - npx htmlhint --version
    - npx htmlhint build/
  rules:
    - if: $CI_PIPELINE_SOURCE == 'merge_request_event'  # Run for all changes to a merge request's source branch
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH       # Run for all changes to the default branch

pages:
  stage: deploy
  dependencies:
    - build-job
  script:
    - mv build/ public/
  artifacts:
    paths:
      - "public/"
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH      # Run for all changes to the default branch only
```

- Click `Commit changes`

- Click `Create merge request`  ***Merge işleminden sonra merge yaptığınız branch'ın silinmesini istiyorsanız, seçeneğin olduğu kare kutucuğa tick atmayı unutmayın, yani bu merge işleminden sonra sadece main branch'ım kalacak. İsterseniz tick atmayın diğer branch da kalsın.***
  " New merge request
    From feature1 into main   Change branches
  `Title (required)` : `Update .gitlab-ci.yml`
  `Description` :   `A merge request from feature1 branch into main branch for updating .gitlab-ci.yml in order to run the jobs and deploy page.`
  `Assignee` : `arrow levent` ***kendi user'ını seç, Unassigned'ı seçme***
  `Reviewer` : `arrow levent` ***kendi user'ını seç, Unassigned'ı seçme***
    Create merge request
  Click `Approve` (Optional)
  Click `Merge`
  
- Pipeline aşamalarını/joblarını buradan gözlemleyebilirsin `Build` -- `Pipelines`

- Static web sitesini görmek için : `https://arrowlevent.gitlab.io/nodejs-project/`


## Part-8 : Start using merge request pipelines

- Reduce duplicated configuration :

  - The pipeline now contains three jobs that all have identical `rules` and `image` configuration. Instead of repeating these rules, use `extends` and `default` to create single sources of truth.

- This Part-8 introduces:

  - `Hidden jobs:` Jobs that start with `.` are never added to a pipeline. Use them to hold configuration you want to reuse. [https://docs.gitlab.com/ee/ci/jobs/index.html#hide-jobs]

  - `extends:` Use extends to repeat configuration in multiple places, often from hidden jobs. If you update the hidden job’s configuration, all jobs extending the hidden job use the updated configuration. [https://docs.gitlab.com/ee/ci/yaml/index.html#extends]

  - `default:` Set keyword defaults that apply to all jobs when not defined. [https://docs.gitlab.com/ee/ci/yaml/index.html#default]

  - `YAML overriding`: When reusing configuration with `extends` or `default`, you can explicitly define a keyword in the job to override the `extends` or `default` configuration.

- In this Part-8:
  - Add a `.standard-rules` hidden job to hold the rules that are repeated in `build-job`, `lint-markdown`, and `test-html`.

  - Use `extends` to reuse the `.standard-rules` configuration in the three jobs.

  - Add a `default` section to define the `image` default as `node`.

  - The `pages` deployment job does not need the default `node` image, so explicitly use `busybox`, an extremely tiny and fast image. [https://hub.docker.com/_/busybox]

- Modify .gitlab-ci.yml

```yaml (.gitlab-ci.yml)
stages:
  - build
  - test
  - deploy

default:               # Add a default section to define the `image` keyword's default value
  image: node

.standard-rules:       # Make a hidden job to hold the common rules
  rules:
    - if: $CI_PIPELINE_SOURCE == 'merge_request_event'
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

build-job:
  extends:
    - .standard-rules  # Reuse the configuration in `.standard-rules` here
  stage: build
  script:
    - npm install
    - npm run build
  artifacts:
    paths:
      - "build/"

lint-markdown:
  stage: test
  extends:
    - .standard-rules  # Reuse the configuration in `.standard-rules` here
  dependencies: []
  script:
    - npm install markdownlint-cli2 --global
    - markdownlint-cli2 -v
    - markdownlint-cli2 "blog/**/*.md" "docs/**/*.md"
  allow_failure: true

test-html:
  stage: test
  extends:
    - .standard-rules  # Reuse the configuration in `.standard-rules` here
  dependencies:
    - build-job
  script:
    - npm install --save-dev htmlhint
    - npx htmlhint --version
    - npx htmlhint build/

pages:
  stage: deploy
  image: busybox       # Override the default `image` value with `busybox`
  dependencies:
    - build-job
  script:
    - mv build/ public/
  artifacts:
    paths:
      - "public/"
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
```
- `Build` -- `Pipelines` Pipeline aşamalarını/joblarını buradan gözlemleyebilirsin

- Static web sitesini görmek için : `https://arrowlevent.gitlab.io/nodejs-project/`


## Resources :

- https://gitlab.com/arrowlevent/nodejs-project   (`My Public GitLab repository`)
- https://docs.gitlab.com/ee/ci/quick_start/tutorial.html
- https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-up-node-on-ec2-instance.html
- https://docs.gitlab.com/ee/user/project/pages/index.html
- https://github.com/DavidAnson/markdownlint
- https://docs.gitlab.com/ee/ci/yaml/?query=dependencies#dependencies
- https://htmlhint.com/
- https://docs.gitlab.com/ee/ci/yaml/index.html#rules
- https://docs.gitlab.com/ee/ci/variables/index.html
- https://docs.gitlab.com/ee/ci/variables/predefined_variables.html
- https://docs.gitlab.com/ee/ci/jobs/index.html#hide-jobs
- https://docs.gitlab.com/ee/ci/yaml/index.html#extends
- https://docs.gitlab.com/ee/ci/yaml/index.html#default
- https://hub.docker.com/_/busybox