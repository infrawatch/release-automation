# Infrawatch Release Automation

Common release and automation tooling for infrawatch project, aka Service Telemetry Framework (STF).

## Related images build process

### Repositories and tag definition for related images

All the related images are stored and automatically build in
https://quay.io/repository/infrawatch.

The related images include:

- https://quay.io/repository/infrawatch/sg-core
- https://quay.io/repository/infrawatch/sg-bridge
- https://quay.io/repository/infrawatch/prometheus-webhook-snmp
- https://quay.io/repository/infrawatch/service-telemetry-operator
- https://quay.io/repository/infrawatch/smart-gateway-operator

In their respective repositories, we differentiate the following
tags:

- nightly
- nightly-1.5

We use those tags to refer to the latest available nightly builds
(built periodically) for the master and stable-1.5 branches
respectively.

We also include the tags for the latest release, e.g. Smart Gateway Operator
(SGO) v5.1.2, and the "latest" and "master" tags (point to the same builds).

Those tags are used when cutting a new release.

### Build triggers for related images

For each of the related images the following build triggers are defined:

- Push to stable-x.y.z branch
- Push to master branch
- Push to tags/v.x.y.z

These actions will generate a new build to be created and stored
in the specific repository.

## Bundles build process

### Repositories and tag definition for bundles

Operator bundles are stored in https://quay.io/organization/infrawatch-operators.

The bundles include:

- https://quay.io/repository/infrawatch-operators/smart-gateway-operator-bundle
- https://quay.io/repository/infrawatch-operators/service-telemetry-operator-bundle

In their respective repositories, we differentiate the following
tags:

- nightly-head
- nightly-1.5

We use those tags to refer to the latest available nightly builds
(built periodically) for the master and stable-1.5 branches
respectively.

### Build process for bundles

Bundles build process is not automatically triggered by the repository
as we do with the related images.

Instead, we rely on the execution of the nightly action defined in the
release-automation repository. Precisely, https://github.com/infrawatch/release-automation/blob/main/.github/workflows/nightly.yml.

This action fetches master and stable-1.5 branches for both Service Telemetry
Operator (STO) and Smart Gateway Operator (SGO), and executes the `releaser.sh`
script in
https://github.com/infrawatch/release-automation/blob/main/releaser.sh.

The `releaser.sh` script automates retrieving the latest available builds for
the related images, setting the correct tags for them, creating the bundles
(more information on this in the section below), replacing tags for correct
image hashes in the created bundles, validating the bundles and pushing the
created bundles to their respective repository.

### Bundle creation

The bundle creation is delegated to logic present in each of the operators repository.

Precisely, we generate the bundle for STO using this script
https://github.com/infrawatch/service-telemetry-operator/blob/master/build/generate_bundle.sh
and we generate the bundle for SGO using this script
https://github.com/infrawatch/smart-gateway-operator/blob/master/build/generate_bundle.sh.
Both work in a very similar way.

The `generate_bundle.sh` script executes the following steps

- Generate version
- Create working directory
- Generate Dockerfile
- Generate bundle
- Copy extra metadata

#### Bundle version

The bundle version is generated by combining the operator CSV major version and
the current UNIX epoch time. For example, stf/service-telemetry-operator-bundle
version=1.5.1723675485, describes a build for STO 1.5, built at epoch time
1723675485, which converts to Aug 14 2024 at 10:45am. You can use
https://www.epochconverter.com/ to get the specific date in which a desired
bundle has been built.

## Index image build process

The creation process of an index image with SGO and STO bundles is also
automated. The index image is stored in
https://quay.io/organization/infrawatch-operators.

Precisely, the latest available index image can be found at:

- https://quay.io/repository/infrawatch-operators/infrawatch-catalog

As with the operator bundles, we differenciate the following tags:

- nightly-head
- nightly-1.5

We use those tags to refer to the latest available nightly builds
(built periodically) for the master and stable-1.5 branches
respectively.

### Build process for the index image

We also rely on the execution of the nightly action defined in the
release-automation repository for the creation of the index image. Precisely,
https://github.com/infrawatch/release-automation/blob/main/.github/workflows/nightly.yml.

This action fetches master and stable-1.5 branches for both STO and SGO, and
executes the releaser.sh script in
https://github.com/infrawatch/release-automation/blob/main/releaser.sh.

After the creation of STO and SGO bundles, a step for building the index image
is executed and the result is pushed to the quay.io container repository.
