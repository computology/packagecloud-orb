# packagecloud orb

CircleCI orb for interacting with packagecloud repositories

## Jobs
* *Push* - Upload packages to a packagecloud repository
* *Yank* - Remove packages from a packagecloud repository

### Push examples

#### Push packages from project root
This example assumes a package exists in the `package-path` directory:
```
version: 2.1
orbs:
  packagecloud: packagecloud/packagecloud@dev:0.0.17

workflows:
  build:
    jobs:
      - packagecloud/push:
          name: "Upload package"
          repo-fqname: "armando/test-orb"
          os-version: "node"
          package-path: "./packages/hello-world-0.0.1.tgz"
```

#### Push packages with build steps
This example shows how you can use the `build-steps` configuration to build a package and push it to a repo:
```
version: 2.1
orbs:
  packagecloud: packagecloud/packagecloud@dev:0.0.17

workflows:
  build:
    jobs:
      - packagecloud/push:
          executor-version: "2.3.8-jessie-node"
          name: "Upload package"
          repo-fqname: "armando/test-orb"
          os-version: "node"
          package-path: "hello-world-0.0.1.tgz"
          build-steps:
            - run: npm pack
```

#### Push packages from a CircleCI workspace
You can also push packages from a workspace by passing the `packagecloud-orb` a workspace path:
```
version: 2.1
orbs:
  packagecloud: packagecloud/packagecloud@dev:0.0.17

defaults: &defaults
  working_directory: ~/repo
  docker:
    - image: circleci/node:8.9.1

jobs:
  test:
    <<: *defaults  
    steps:
      - checkout

      - restore_cache:
          keys:
          - v1-dependencies-{{ checksum "package.json" }}
          - v1-dependencies-

      - run: npm install
      - run:
          name: Run tests
          command: npm test

      - run:
          name: Build package
          command: npm pack

      - save_cache:
          paths:
            - node_modules
          key: v1-dependencies-{{ checksum "package.json" }}

      - persist_to_workspace:
          root: ~/repo
          paths: .
            
workflows:
  build:
    jobs:
      - test
      - packagecloud/push:
          repo-fqname: "armando/test-orb"
          os-version: "node"
          package-path: "./hello-world-0.1.3.tgz"
          workspace-path: "."
          requires:
            - test
```

### Yank examples

#### Yank a package from a packagecloud repository

```
version: 2.1
orbs:
  packagecloud: packagecloud/packagecloud@dev:0.0.17

workflows:
  build:
    jobs:
      - packagecloud/yank:
          repo-fqname: "armando/test-orb"
          os-version: "node"
          package-name: "hello-world-0.1.3.tgz"

```

## Parameters
### packagecloud/push

| Parameter        | type    | default    |    description |
|------------------|--------|-------------|----------------|
| executor-version | string |  'circleci/ruby:2.3-jessie’ | Docker image to use with the packagecloud CLI |
| repo-fqname | string |  '' | *Required:* Fully-qualified path to a packagecloud repository -- e.g., example_user/example_repo |
| os-version | string | '' | *Required:* Version string associated with a package for pushing/yanking -- e.g., 'ubuntu/bionic' [https://packagecloud.io/docs#os_distro_version](https://packagecloud.io/docs#￼os_distro_version) |
| package-path | string | '' | *Required:* Path to package(s) to upload into a packagecloud repository. The CLI also accepts a glob pattern for packages — e.g., ./path/*.deb |
| build-steps | steps |  [] | Steps necessary to build packages. Alternately provide `workspace-path` |
| workspace-path | string |  '' | The key of a workflow workspace which contains packages. Alternately provide `build-steps` |

### packagecloud/yank

| Parameter        | type    | default    |    description |
|------------------|--------|-------------|----------------|
| repo-fqname | string |  '' | *Required:* Fully-qualified path to a packagecloud repository -- e.g., example_user/example_repo |
| os-version | string | '' | *Required:* Version string associated with a package for yanking -- e.g., 'ubuntu/bionic' [https://packagecloud.io/docs#os_distro_version](https://packagecloud.io/docs#￼os_distro_version) |
| package-name | string | '' | *Required:* Package name to yank from repository. When yanking a package, pass the full name of the package -- e.g., 'coolpackage_2.1.1_amd64.deb' |
