### GitLab CI Basic Notes for Pipeline

- `.gitlab-ci.yml` file is used to define a pipeline.
- .`yaml` is a data serialization language used by GitLab CI.
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

### GitLab Script Commands

|      Script Commands       |                                                                             Description                                                                             |
| :------------------------: | :-----------------------------------------------------------------------------------------------------------------------------------------------------------------: |
|    `test -f <filename>`    |     `test` command is used to verify that the file car.txt was created. <br> `-f` flag is needed to check that the specified file exists and is a regular file.     |
| `grep "wheels" <filename>` | `grep` command is used for searching lines that match a regular expression. <br> It does a global search with the regular expresison and prints all matching lines. |
|      `echo "text" >>`      |                                                                          to append content                                                                          |
|      `echo "text" >`       |                                                                         to replace content                                                                          |
