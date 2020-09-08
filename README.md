# circleci-security
CircleCI Orb that allows you to leverage multiple security tools in your pipeline. \
Invoking any of the security steps will run the relevant tool, and upload its output (in JSON format) as a CircleCi artifact.

# Current Version
*salidas/security@0.3.0*

# Table of Contents

  * [Usage / Installation](#usage--installation)
  * [Example Usage](#example-usage-in-a-circleciconfigyml-file)
  * [Features - Individual Security Tools](#features---individual-security-tools)
      * [JavaScript](#javascript)
        * [insider](#insider)
        * [Snyk](#snyk)
      * [Golang](#golang)
        * [gosec](#gosec)
        * [nancy](#nancy)
      * [Secrets](#secrets)
        * [gitleaks](#gitleaks)
      * [Containers](#containers)
        * [trivy](#trivy)
      * [HTTP Headers and Secrets](#http-headers-and-cookies)
        * [SHeD](#shed)

# Usage / Installation
Add the following orb to your .circleci/config.yml file:
```yaml
orbs:
  security: salidas/security@0.2.0
```

Then, add security steps into your existing CircleCI build jobs! \
See **Features** below for a list of steps you can call.

# Example Usage in a .circleci/config.yml file
```yaml
version: 2.1

orbs:
  security: salidas/security@0.2.0

jobs:
  check_node_dependencies: # The job you create/use to build/pull/test your project
    docker:
      - image: circleci/node:buster
    steps:
      - checkout
      - run: # Standard Node project step
          name: "[check_node_dependencies][Install] Run 'yarn install'"
          command: |
            CIRCLE_WORKING_DIRECTORY="${CIRCLE_WORKING_DIRECTORY/#\~/$HOME}"
            cd $CIRCLE_WORKING_DIRECTORY
            yarn install
      - security/dependencies_snyk_node # Insert the security steps you wish to run
      
 workflows:
  build:
    jobs:
      - check_node_dependencies
```

# Features - Individual Security Tools
The following are CircleCI build steps you can add to your existing build processes:
## JavaScript
### insider
Uses [insider-cli](https://github.com/insidersec/insider) to check JavaScript code for security issues.
#### Parameters
- (optional) `target_directory` - The location of the project to be scanned. By default this is "~/project", which is also where CircleCI typically clones to.
#### Usage
```yaml
- security/javascript_insider
```

### Snyk
Uses Snyk to check your Node project's dependencies for any publicly known vulnerabilities.
#### Requirements
- An environment variable containing your SNYK API key. The step will fail without this! By default Snyk looks for the value of SNYK_TOKEN.
#### Parameters
- (optional) `token` - The name of the environment variable storing your SNYK API key. By default this is SNYK_TOKEN.
- (optional) `target_directory` - The location of the project to be scanned. By default this is "~/project", which is also where CircleCI typically clones to.
#### Usage
```yaml
- security/dependencies_snyk_node
```

## Golang
### gosec
Uses [gosec](https://github.com/securego/gosec) to check code for potential security issues.
#### Parameters
- (optional) `target_directory` - The location of the project to be scanned. By default this is "~/project", which is also where CircleCI typically clones to.
#### Usage
```yaml
- security/golang_gosec
```

### nancy
Uses [nancy](https://github.com/sonatype-nexus-community/nancy) to leverage Sonatype's OSS database and look for Golang dependency issues/vulnerabilities.
#### Parameters
- (optional) `target_directory` - The location of the project to be scanned. By default this is "~/project", which is also where CircleCI typically clones to.
#### Usage
```yaml
- security/golang_nancy
```

## Secrets
### gitleaks
Uses [gitleaks](https://github.com/zricethezav/gitleaks) to scan the checked out repository for potentially hardcoded secrets and keys.
#### Parameters
- (optional) `target_directory` - The location of the project to be scanned. By default this is "~/project", which is also where CircleCI typically clones to.
#### Usage
```yaml
- security/secrets_gitleaks
```

## Containers
### trivy
Uses [trivy](https://github.com/aquasecurity/trivy) to scan an image for vulnerabilities in its dependencies/packages.
#### Parameters
- (required) `image_name` - The name of the Docker image to be pulled and scanned.
#### Usage
```yaml
- security/container_trivy:
    image_name: vulnerables/web-dvwa
```

## HTTP Headers and Cookies
### SHeD
Runs [a tool I created](https://github.com/itsdean/shed) to target a URL and provide information on security headers returned, \
along with any cookies and their enabled/disabled/missing flags.
#### Parameters
- (required) `url` - The target URL to be accessed. It should be in FQDN format, i.e. https://foo.bar/bridge/to/terabithia
- (optional) `headers` - Any headers to be provided in the request, in "a: x|b: y" format (where the pipe delimits a new header to be passed)
#### Usage
```yaml
- security/shed:
    url: "http://127.0.0.1"
    headers: "Cookie: a=x;b=y;|Authorisation: Basic foobar"
```
