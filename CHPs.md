# Container Hardening Priorities (CHPs)

This document describes a method of classifying container images in terms of "security hardening"
levels. 

As an industry, we often talk about container images being "security hardened" or one image being
more secure than another. To date, this has been loosely defined and subjective. This document
attempts an objective classification, enabling comparisons of images and identification of areas for
potential improvement. 

The hope is that this document becomes the beginning of something larger and, to that end, we
encourage discussion and contributions from the community. 

As with any document of this sort, it is impossible to guarantee security and all advice should be
analyzed from your own context. That said, we hope the document is useful in guiding teams to better
practices.

## Audience

The primary audience is DevOps teams trying to assess the security standing of their image
building process, but it can also be used to grade and compare images from third parties.

Anyone building container images is encouraged to assess their builds according to this document and
publish the results.

## Scope

Scope of this document is limited to OCI Container image development, particularly during build pipelines.
We do not cover runtime configuration, which may be covered a separate document in the
future. For the purposes of this document we assume that the image author is trusted and will not
intentionally insert malware. Software supply chain security is a large focus of this document;
while the original author is considered trusted, details of how images are distributed and verified
are in scope. Application specific vulnerabilities (e.g. buffer overflows or malware in application code) are not
considered although some of the criteria here may reduce the impact of such vulnerabilities.

The guidance is intended to be applicable to all container build systems, but where specific
advice is given, it is generally assumed Dockerfiles are being used. People using other systems
should be easily able to adapt the advice.

## Security Vectors

This document defines several "vectors" across which container images can be judged. These are:

 - **Minimalism**

A more "minimal" image will contain less software than a comparative image. Having less
software is useful in terms of reducing "attack surface". If a software package is not present, it
cannot be leveraged in an attack. Note that [image
complexity](https://www.chainguard.dev/unchained/image-sizes-miss-the-point) in terms of number of
packages installed is a better measure of minimalism than raw image size.

Minimalism becomes particularly important in "living off the land" attacks where trusted, legitimate
tooling is leveraged by the attackers. Reducing the availability of these tools makes attackers'
lives harder. 

 - **Provenance**

Provenance refers to the origin or source of something. In software, we can *prove* the provenance
of artifacts with techniques including cryptographic signatures and checksums. Proving provenance is
the cornerstone of software supply chain security; it is essential to know where your software comes
from and that you are not accidentally running a malicious or tampered version. Note that provenance
extends beyond where container images come from; we also need to consider where all the software
that goes into the container image came from.

 - **Configuration and Metadata**

There are various configuration settings and patterns for improving container security. For example,
[images should always define a user other than root to run as](#user-is-non-root).

 - **Vulnerabilities**

Vulnerabilities are exploitable flaws in software. A major goal of security is to reduce the
number of vulnerabilities and their potential impact as much as possible. 

Publicly identified vulnerabilities are are often assigned a Common Vulnerabilities and Exposure
(CVE) identifier on a database such as [National Vulnerability Database
(NVD)](https://nvd.nist.gov/). CVEs are graded from critical to negligible according to perceived
impact. We recommend using tooling (commonly called vulnerability scanners) to check software and
container images for the presence of known  CVEs. Readers should not autaomtically dismiss lower
graded CVES, due to the potential impact of _CVE chaining_, where multiple, possibly low impact
vulnerabilities are exploited in combination to achieve a higher-impact attack. 

Because our knowledge of CVEs is constantly evolving, container images can only be graded on this
vector at a point in time; the same image may be CVE free one week but not the next.

## Levels

Providing a long list of suggestions for security hardening risks being overwhelming and
demotivating. In order to mitigate this risk, the advice has been broken into 5 levels. This means
that people new to security can start by working on level 1, without worrying about higher levels
until they have understood and addressed simpler issues. The highest levels are intended to
be aspirational, with few container images meeting this standard.

It is likely that many container images will meet some items at various levels but not all. In
terms of the vulnerability vector, the level will change over time. So a newly built image may
have no known CVEs (level 5) but next week a new CVE could reduce that level.

Readers should feel empowered to use the advice in this document as best works for them and their
organization.

The suggestions are intended to be objectively checkable, so images can be easily (and potentially
automatically) graded. This provides a simple way to compare images against each other, but any
comparisons should always be considered in context; an image that meets all the criteria here may
have security deficiencies not present in an image that only meets a few criteria. This list is not
exhaustive and could not be exhaustive, always practice [defence in
depth](https://en.wikipedia.org/wiki/Defense_in_depth_(computing)) and follow the [principle of
least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege).

In keeping with the salsa and chips theme, the levels have been named after chillis on various
levels of the Scoville scale.

## Summary Table

| Level | Minimalism | Provenance | Configuration & Metadata | Vulnerability |
| ----- | ---------- | ---------- | ------------------------ | --------- |
| **1 Pimiento** | • Use minimal base image | • Download from trusted source |  • No secrets in image | |
| | | | | |
| **2 Jalapeno** | • No build or debug tooling | • Verify d/ls by sig or checksum | • No files with elevated privileges |• No critical |
| | | • Sign images | • User is nonroot | |
| | | • Image recently updated | | |
| | | | | |
| **3 Scotch Bonnet** | • No shell | • Pin images to digests | • Support secrets via file mount  | • No high |
| | • No package manager | • Pin packages | • Add annotations | |
| | | | | |
| **4 Nagabon** | | • Provenance attestations | | • No medium |
| | | • SBOMs | | |
| | | | | |
| **5 Ghost** | | • Fully reproducible | • Security profile in metadata | • No known CVEs |

## Badges

One of the goals of this project is to provide users with an simple way to differentiate images. As
part of that goal we encourage the use of _badges_ on READMEs and other documentation to describe
where your images are on the CHPs scale e.g:

![CHPs Score](https://img.shields.io/badge/overall-A%2B-gold?style=flat-square&labelColor=%233443F4&color=%2301A178)
![Minimalism](https://img.shields.io/badge/minimalism-A%2B-gold?style=flat-square&labelColor=%233443F4&color=%2301A178)
![Provenance](https://img.shields.io/badge/provenance-A-gold?style=flat-square&labelColor=%233443F4&color=%2381FEA0)
![Config](https://img.shields.io/badge/config-C-gold?style=flat-square&labelColor=%233443F4&color=%23F6EB61)
![Vulns](https://img.shields.io/badge/CVEs-D-gold?style=flat-square&labelColor=%233443F4&color=%23FE5B3C)

There is a grade per vector and an overall grade. The grades are determined by assigning a point for
each criteria the image meets. This isn't a perfect system, as not all criteria are "equal". We
welcome suggestions for improving the system.

Note that grades are per image, so a single repository may have multiple images with different tags
e.g. `redis:latest`, `redis:latest-slim`, `redis:latest-debug`. The badges provide a fast, visual way to
identify differences between tags. For example a "debug" image may score lower on the minimalism scale
indicating you may want to use a different tag when deploying to production.

Grades are ranked according to the percentage of criteria achieved within the relevant vector:

| %   | Grade |
| --- | ----- |
| 100 | A+ |
| 75  | A |
| 50  | B |
| 40  | C |
| -   | D |

At the moment grading is a self-scoring honor system, but there is scope to create an automated grading
system with an API that analyses images and returns a badge.

## Discussion by Vector

Each of the items in the table is discussed in more depth below. Note grouping is by vector rather than level.

### Minimalism

These items all reduce the amount of software packages and complexity in the image.

#### Use a Minimal Base Image

Many container base images contain a large set of Linux packages for usability purposes. This is
very helpful when learning containers and in development, but production images should only
contain the packages required by the application being containerized. For example, tools like
curl or wget are very useful for obtaining software in development, but can also be used by an
attacker to obtain malicious tooling or access other parts of the system. 

For this reason, production images should use a slimmed down image such as `debian:slim`, `alpine`
or a distroless image from Chainguard or Google.

#### No Build or Debug Tooling.

When building an image, it's common to have a compile step or similar, which requires build tooling
to be present in the image. This tooling should not be present in the final image. This can be
achieved by using a [multi-stage build](https://docs.docker.com/build/building/multi-stage/) where
the built assets are copied onto a new image. 

The same is true for debug tooling. You may have an image build that enables debug tooling such as
debuggers or tracing tools. These options should be turned off in production builds and where
possible the tools should be removed from the image. 

Note that you may well have images breaking this rule in development, but they shouldn't run in
production clusters.

#### No Shell

Shells are very useful when developing and debugging images, but they are rarely required for
production images and can provide significant resources to an attacker.

Removing shells does mean that the traditional debugging technique of `docker exec -it
container_name sh` (and equivalents such as `kubectl exec`) will not work. In these cases you may
want to look at alternative tooling, such as [docker
debug](https://docs.docker.com/reference/cli/docker/debug/), [kubectl
debug](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_debug/) or
[cdebug](https://github.com/iximiuz/cdebug) that can start a temporary process with a shell for
debugging. 

Scripts are also often used in entrypoint scripts and healthchecks. You may be able to replace
entrypoint scripts with a separate and temporary [init
container](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) that contains the
tooling, otherwise you will need to replace the script with a compiled binary.

#### No Package Manager

Package managers are used to install extra dependencies and tooling. In a production image this will
already have happened, so there is no call for a package manager and removing it can potentially
make attackers have to work harder.

As the package manager is required by most distribution base images, removing the package manager
typically requires using a multistage build where the final production image runs on a distroless
image, such as [Chainguard's static
image](https://images.chainguard.dev/directory/image/static/overview) or [Google's
distroless](https://github.com/GoogleContainerTools/distroless). 


### Provenance

These items all improve supply-chain security; knowing and proving where the software comes from and
how it was built.

#### Download From Trusted Source

It's important to always be aware of who has created the images you're using. In the case of Docker
Hub images, is it an official image? Or a verified publisher? Is the image getting updates? Use
extra caution when pulling images from user repositories with no level of checking or oversight.
Only use images that are being actively maintained.

#### Verify Downloads

Nearly all container builds will download or install third party software. If installed via the
package manager (e.g. apt, apk, yum) it will be automatically tested against a checksum to ensure it
hasn't been tampered with. Software downloaded directly (e.g. via `wget` or `curl`) needs separate
checks. This may mean checking signatures, or using a checksum. This will ensure that the artifact
has not been tampered with or replaced. The [Docker official images
guidance](https://github.com/docker-library/official-images?tab=readme-ov-file#security) includes
more information on how to do this correctly.

Note that the `ADD` statement in Dockerfiles can be used to download software and supports a
[checksum](https://docs.docker.com/reference/dockerfile/#add---checksum) option. This can be useful
in cases where wget or curl is not available in the build image. `ADD` should only be used for remote
files; use `COPY` when adding local files into images.

#### Sign Images

Signing container images allows others to verify the origin and authenticity of images and ensures
they haven't been tampered with.

There are multiple solutions for signing container images, including
[sigstore](https://www.sigstore.dev/) and [notation](https://github.com/notaryproject/notation).
[Keyless signing](https://docs.sigstore.dev/cosign/signing/overview/) means that there is no need to
store a private key, significantly simplifying the set-up and maintenance burden of signing. New
users can refer to this [walkthrough on how to sign an image with cosign](https://edu.chainguard.dev/open-source/sigstore/cosign/how-to-sign-a-container-with-cosign/).

For signing to be effective, signatures must be verified at some point. This can be done in
with CLI tools like [cosign](https://github.com/sigstore/cosign) or platform specific integrations such as 
[policy-controller](https://docs.sigstore.dev/policy-controller/overview/) for Kubernetes.

#### Image Recently Updated

Images should be regularly rebuilt to ensure they are up-to-date with the latest security fixes.
Even when the code for the main application hasn't changed, it is likely than transitive
dependencies or other software present in the image has.

Images that have not been updated for an extended period are highly likely to have multiple serious
known vulnerabilities. As a rule of thumb, we would recommend checking that all 3rd party images
have been built in the last 30 days. Anything longer than this suggests that the image is
sporadically maintained. For images that are built within an organisation it should be more often
e.g. weekly or even daily.

This advice has overlap with the concept of "[build
horizon](https://www.chainguard.dev/unchained/conquering-your-build-horizon)", a practice whereby
assets running in production built before a certain date (the horizon) are automatically rebuilt and
redeployed.

#### Pin Images to Digests

Images are normally built on top of other existing images e.g:

```Dockerfile
FROM cgr.dev/chainguard/wolfi-base:latest
RUN ...
```

In these cases, it is advisable to pin the image version to a digest. The digest is a content based
hash of the image. If the image changes, the digest will change. This means that rerunning the build
will use exactly the same base image, no matter where the build is done or who performs it. As well
as having benefits for reproducibility, this also reduces the scope for an attacker to replace an
image with a compromised equivalent.

Pinning to a digest looks like this:

```Dockerfile
FROM cgr.dev/chainguard/wolfi-base:latest@sha256:52f88fede0eba350de7be98a4a803be5072e5ddcd8b5c7226d3ebbcd126fb388
RUN apk update ...
```

The downside of pinning images to digests is that it becomes more work to update them; you don't
automatically get a new version or bug-fixes by just rerunning the build. To help automate work like
this, take a look at tools like [frizbee](https://github.com/stacklok/frizbee) and
[digestabot](https://github.com/chainguard-dev/digestabot/).

#### Pin Packages

Builds will often install software from a package manager e.g:

```
RUN apk update && apk install curl

```

In these cases it's advisable to pin the image to the extent supported by the package manager e.g:

```
RUN apk update && apk install curl=8.11.1-r0

```

Note that in some cases the distribution will not keep old builds of packages, meaning that your
build may break when the package is updated. In these cases, you will need to maintain a package
mirror to guarantee the ability to rebuild old versions of images.

Again there is a downside to pinning; you will need to regularly update the pins when rebuilding to get new versions and bugfixes. 

#### SLSA Provenance Attestations

Attestations are verifiable claims related to a software artifact, especially in relation to how it
was produced. These are typically created via the [in-toto framework](https://in-toto.io/), which
defines both the format for attestations and provides a workflow for their creation and
verification. Image builders are encouraged to produce a wide range of attestations, some of which
may be bespoke to a given use case.

[SLSA Provenance](https://slsa.dev/provenance/v1) defines a standard format that details how an
artifact was built. For example, this will typically include where the source code came from, which
compiler was used with what arguments, when the build started and finished.

One way to attach in-toto attestations to a container is by using
[cosign](https://docs.sigstore.dev/cosign/) from the Sigstore project.

#### SBOMs

[SBOMs](https://en.wikipedia.org/wiki/Software_Bill_of_Materials) or Software Bill of Materials are
a complete list of all the software inside an artifact. An SBOM should specify all the libraries
used and their exact versions. This allows users to determine exactly what is in the container,
which is useful in multiple scenarios, but particularly when assessing organizational exposure to known
vulnerabilities.

The two most common formats for SBOMs are [SPDX](https://spdx.dev/) and [CycloneDX](https://cyclonedx.org/).
SBOMs are typically linked to a container via an attestation.

#### Full Reproducibility

Full reproducibility means that if a build is re-run, the built artifacts will be identical at
the bit level. This is significantly beneficial for supply-chain security as it makes is possible to
prove that an artifact was not tampered with at the build stage.

Getting to this level of reproducibility with containers is difficult -- a lot of build tooling such
as `docker build` will insert timestamps or unique IDs that break full reproducibility. Projects
that claim success in this space include [apko](https://github.com/chainguard-dev/apko),
[bazel](https://bazel.build/) and
[nix](https://nix.dev/tutorials/nixos/building-and-running-docker-images.html).

### Configuration and Metadata

This vector covers various advice on how to configure Docker Images for security. In many cases this
means creating or configuring metadata that can be applied at runtime. 

#### No Secrets

Never store secrets in container images. Anyone with access to the image will be able to extract
them. For applications that require secrets at runtime, these should be passed in secure manner when
the container is started. There are multiple solutions for this, but a common technique is to mount
secrets in volumes at runtime. There are also complete enterprise solutions such as the External
Secrets Operator and Vault.

For secrets that are needed at build time, for example in order to access a download, the container
build tooling should provide a secure feature, for example, see [Docker Build
Secrets](https://docs.docker.com/build/building/secrets/)

You can check images (and other artifacts) for the presence of secrets via tools such as
[trufflehog](https://github.com/trufflesecurity/trufflehog.)

#### No files with elevated privileges

There are two ways in which files can be assigned elevated privileges; by setting the setuid bit
(creating an SUID executable), or by setting capabilities on the file. 

An SUID executable operates with the file owner’s privileges rather than the invoking user’s. This
allows programs to execute with elevated privileges, typically those of the root user. If an
attacker is able to exploit a vulnerability in an SUID executable they may be able to increase their
privileges inside a container.

Production containers typically do not require SUID executables and they should not be present. SUID
executables can be easily identified e.g. by running this find command:

```
find / -perm -u=s -type f 2>/dev/null
```

In cases where an executable inside a production container does require elevated privileges,
capabilities should be used instead. In the same vein, no files should have elevated capabilities
unless required by the production application. Files with elevated capabilities can be identified
with a similar command, assuming the `getcap` utility is installed:

```
find / -type f -exec getcap '{}' + 2>/dev/null
```

#### User is non-root

Containers which run as root have elevated access privileges to files and processes in a container.
Should the container or application be compromised, the attacker will gain these raised privileges.
Worse, if the attacker manages to break-out from the container, they will have root privileges on
the host (assuming no other preventions are in place such as user namespacing). For these reasons it
is important that containers do not run as root in production.

Base images are an exception to this rule as they will typically run as root to allow users
to install software with the package manager. However, base images should be only used in
build workflows and testing; they should not run directly in production.

Some workflows may require a container to start as root to perform an action such as changing file
permissions. Where possible this action should be moved to an init container. If this is not
possible, the user should be changed to an unprivileged user immediately after performing the
action.

#### Support Secrets via File

The classic 12-factor app manifesto advocated for storing configuration in environment variables. This led
to a lot of container images supporting secrets being passed in this manner. This is generally
[considered bad advice](https://blog.arcjet.com/storing-secrets-in-env-vars-considered-harmful/)
now, as they are easily exposed (for example in log output).

Where possible, containers should support passing secrets by file instead. For example, see the
[postgres official image](https://hub.docker.com/_/postgres/) which supports setting the
`POSTGRES_PASSWORD_FILE` variable, which should point to a file mounted at runtime.

This can be harder to achieve when containerising third-party software, but consider if you can
create a wrapper script the reads the secret file and passes it to the application in whatever
format it expects.

#### Add Annotations

The [OCI Image Specification](https://github.com/opencontainers/image-spec/blob/main/annotations.md) defines several [annotations](https://github.com/opencontainers/image-spec/blob/main/annotations.md) which are used to describe features of images in a standard way. Which annotations to use depends on the exact image in question, but common ones include:

  - `org.opencontainers.image.authors` for defining the author of an image (often a company name,
    although `vendor` is also used for this purpose)
  - `org.opencontainers.image.created` which should contain a timestamp of when the image was built
  - `org.opencontainers.image.source` for holding a link to any source repository for the image (e.g. GitHub)

These annotations can help users identify images and use them correctly, as well as enabling
automation (e.g. indexing the source repository of all images running in a cluster).

#### Attach a Security Profile

There are multiple Linux tools (such as kernel security modules) for restricting system calls,
primarily [seccomp](https://docs.docker.com/engine/security/seccomp/),
[SELinux](https://en.wikipedia.org/wiki/Security-Enhanced_Linux) and
[AppArmor](https://apparmor.net/). These tools work on "profiles" which determine the actions a
program is allowed to take. A default profile can be (and often is) applied to all containers, but
it is also possible to create container specific profiles -- so if a container doesn't need to make
network connections or change file permissions, it can be prevented from doing so.

Creating a security profile is a non-trivial task, but potentially worthwhile due to the extra
containment and adherence to the least-privilege principle. Profiles can be associated with
container images via attestations.

### Vulnerabilities

This vector is concerned with the number of known
[CVEs](https://en.wikipedia.org/wiki/Common_Vulnerabilities_and_Exposures) in the image. These are
normally discovered with an automated tool such as Grype, Trivy or Docker Scout that will "scan" a
container image and return a list of suspected CVEs. There are both Open Source and commercial tools
available. CVEs are normally graded according to the
[CVSS](https://en.wikipedia.org/wiki/Common_Vulnerability_Scoring_System) standard and assigned a
relevant severity: Critical, High, Medium or Low.

The resultant CVEs can then be investigated and if a CVE is found to be a false-positive or not a
matter of concern, it can be filtered out using a VEX (Vulnerability Exploitability eXchange)
document. There are [tools](https://github.com/openvex/vexctl) available to help build VEX
documents which can be associated with container images via attestations.

The grading here is only for a point-in-time; new CVEs are constantly being discovered over time, so
the same container image is likely to report more vulnerabilities when scanned at a later date.

#### No critical

The image contains no-known critical CVEs.

#### No high

The image contains no-known high CVEs.

#### No medium

The image contains no-known high CVEs.

#### No CVEs

The image contains no-known CVEs.

## Thanks

Thanks to:
 - Dustin Kirkland, who came up with the original concept and list. 
 - Duffie Cooley, Brandon Mitchell, Rory McCune, Jon Johnson, Kyle Crane and Jason Hall for feedback.
