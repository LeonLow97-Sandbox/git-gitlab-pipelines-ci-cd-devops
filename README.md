# Content

<a href="#gitlab commands">GitLab Commands</a>

# Introduction to GitLab

### GitLab CI Basic Notes for Pipeline

- `.gitlab-ci.yml` file is used to **define a pipeline**.
- `.yaml` is a data serialization standard for all programming languages, and is used by GitLab.
- A job contains at least 1 script that will be executed.
- Pipeline will only work if `.gitlab-ci.yml` of the repository.
- Gitlab CI will automatically assign jobs to the "test" stage, even if no test stage was defined.
- Files created or modified by **one job** will NOT be commited in the repository.
- **GitLab Runner** is the tool responsible for actually running the commands defined in a job.

### GitLab Pipeline

- BUILD --> TEST

```
# build jobs in the same stage run in parallel
# Jobs in the next stage run after the jobs from the previous stage complete successfully.
stages:
    - build
    - test

# job in pipeline
build the car:
    stage: build
    script:
        - mkdir build
        - cd build
        - echo "chassis" >> car.txt
        - echo "engine" >> car.txt
        - echo "wheels" >> car.txt
    # artifact repository (keep after process)
    artifacts:
        paths:
            # save the entire folder
            - build/

test the car:
    stage: test
    script:
        # test the existence of the file
        - test -f build/car.txt
        - cd build
        # grep : search for a text in the file
        - cat car.txt
        - grep "chassis" car.txt
        - grep "engine" car.txt
        - grep "wheels" car.txt

```

### Why GitLab CI?

- Simple, Scalable architecture.
- Docker first approach.
- Pipeline as code.
- Merge requests with CI support.

### Exit Code

- `0` : script marked as successful by GitLab Runner (`Job Succeeded`)
- `1 - 255` : job failed. (`ERROR: Job failed: exit code 1`)

### Debugging

- Use `ls` and `cat <filename>` for debugging.

# Basic CI/CD workflow with GitLab CI

### Continuous Integration (CI)

- Integrating code with other developers.
- Practice of continuously integrating code changes.
- Ensures that the project can still be built/compiled.
- Ensures that any changes **pass all tests**, guidelines, and code compliance standards.
- Code changes are integrated automatically.

### Advantages of Continuous Integration (CI)

- Errors are detected early in the development process.
- Reduces integration problems.
- Allows developers to work faster. (detects bugs, errors during the integration process.)

### Continuous Delivery (CD) 

- Ensures that the software can be **deployed anytime to production**.
- Commonly, the latest verison is deployed to a **testing or staging system**.

### Advantages of Continuous Delivery and Continuous Deployment (CD)

- **Ensures that every change is releasable** by testing that it can be deployed.
- Reduced risk of a new deployment.
- Delivers value much faster to the market (competitive advantage, happy customers).

### The Build Step

- Most projects have a **production build step**.
- Developers write code in human readable programming languages (like C, Java, Ruby, JavaScript).
- The **source code needs to be processed further so it can be deployed** (for example, to a production server).

### Docker for Continuous Integration Servers

- Modern CI Server that can run on 2 different versions of E.g., nodeJS 8 and nodeJS 10.
- Specify the use of a specific image.

### Why do Jobs Fail?

<h1 id="gitlab commands">GitLab Commands</h1>

|      Script Commands       |                                                                             Description                                                                             |
| :------------------------: | :-----------------------------------------------------------------------------------------------------------------------------------------------------------------: |
|    `test -f <filename>`    |     `test` command is used to verify that the file car.txt was created. <br> `-f` flag is needed to check that the specified file exists and is a regular file.     |
| `grep "wheels" <filename>` | `grep` command is used for searching lines that match a regular expression. <br> It does a global search with the regular expresison and prints all matching lines. |
|      `echo "text" >>`      |                                                                          to append content                                                                          |
|      `echo "text" >`       |                                                                         to replace content                                                                          |
