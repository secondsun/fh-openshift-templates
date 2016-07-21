# fh-openshift-templates

## Building
```shell
$ npm install
```

Produce a tar with the templates:

```shell
$ grunt
```
The produced tar will be in the ```dist``` directory.

## Testing
To run tests on all templates, run
```shell
$ npm run test
```

To run tests on a specific group of templates or an individual template pass in a `--suite` value
to the tests.
```shell
$ npm run test --suite=generated # Runs tests on all templates

$ npm run test --suite=aaa # Runs tests on all fh-aaa template
```

The list of available template suites which can be passed into `npm run test` can
be found in files within the `suites/` directory. For example, the `backend.js` file
contains the key `aaa`, this relates to the `fh-aaa-single-template.json` file
and specifies some configuration options. To run the tests on the `fh-aaa` template
just run `npm run test` with the `aaa` key specified as the `suite`. E.g.
```shell
npm run test --suite=aaa
```
This can be done with any key in any module in the `suites/` directory.


## Workflow Guidelines

* Use the parameters section to define key values and then reference them in the rest of the template.
* Keep version in template and package.json in sync.

## Productization procedure

Engineering will produce builds frequently, and update the master branch with these.
Productization will have new images less frequently, and will follow this flow:

### fh-openshift-templates

* Create and push a branch at the tip of master, named with the productization build

    * E.g. `RHMAP-4.0.0-DR9`

* Modify the template to include productized images

* Modify the version in template and package.json to a productized version

* Create a pull request in Github against the already created branch

    * This will generate a new build, and publish corresponding artifact on S3

### fhcap

* Create a corresponding branch in fhcap that a PR will be created against

    * Either from master, or from a relevant release branch

* Use [fhcap-cli](https://github.com/fheng/fhcap-cli) to do a component update for fh-openshift-templates

    * E.g. `fhcap component update fh-openshift-templates <version> <buildnumber>`

    The version and build number should correspond to the build on Jenkins.

* Create a pull request on Github against the new branch

* If part of a release, notify @jasonmadigan (engineering release manager)


## setup a local core

Note: When using scripts on mac ssed tool should be installed. By default sed command is used for different purpose:

    brew install ssed
    alias sed='ssed'

* Ensure you're logged in to OpenShift with `oc` inside the `rhel_openshift3` VM, then paste the following (change the value of the project name in the first line if necessary):

``` shell
export CORE_PROJECT_NAME=core
oc new-project $CORE_PROJECT_NAME
cd /mnt/src/fh-openshift-templates/scripts/core

# Set up Secrets
# Note: On the Mac, you will need to replace the `sed` command in scripts/core/secrets.sh with `ssed`
# Note: On the Mac, you will need to add a folder path to the `mktemp` command in scripts/core/secrets.sh
./prerequisites.sh

# Create new SecurityContextConstraint that allows `chroot` capability
sudo oc create -f ../../gitlab-shell/scc-anyuid-with-chroot.json

# Add the default ServiceAccount for the project to that SCC for gitlab-shell (sshd)
sudo oc adm policy add-scc-to-user anyuid-with-chroot \
    system:serviceaccount:${CORE_PROJECT_NAME}:default

./infra.sh
```

* Wait until all the Pods are running, and the MongoDB initiator Pod has exited successfully, then continue with:

``` shell
./backend.sh
```

* Wait until all of the new Pods are running successfully, then continue with:

``` shell
./frontend.sh
```

* After this, and all Pods are successfully running, you should be able to login to http://rhmap.local.feedhenry.io as `rhmap-admin@example.com`
