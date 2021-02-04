---
title: "In-Pipeline Image Analysis"
linkTitle: "In-Pipeline Scanning"
weight: 4 
---

## Pipeline Image Analysis and Scanning

Anchore now supports analysis of images at build time with no requirement to push images up to a registry in order for them to be analyzed and added to the system.

This feature works by executing Syft inside your pipeline and giving it an endpoint and credentials to upload the results to and Anchore deployment. It will analyze the image
locally for package artifacts and upload the analysis and container metadata to Anchore. The system then loads the result after which the image analysis is available for vulnerability queries
and policy evaluations using the Anchore CLI or direct API operations.

The analysis import is processed by the analyzer services, so you will see the image enter the `not_analyzed` state when first uploaded, then `analyzing` and `analyzed`. Once in the `analyzing` state the proces
is usually very fast (seconds) since it only is operating on the provided package manifest rather than having to pull any image data or perform significant IO to unpack an image.

**Important: In 3.0, Syft can only scan for packages (rpms, dpks, npms, gems, jars, and others, but *not* including NuGet packages or Windows container support) but does not perform the deeper filesystem analysis that the Anchore Analyzers do, for example malware scanning, so 
the policy check functionality is more limited since there is less analysis data. However, vulnerability checks and vulnerability policies are full featured.

### Example

```
❯ syft ubuntu:latest -u admin -p foobar -H http://localhost:8228
 ✔ Loaded image
 ✔ Parsed image
 ✔ Cataloged image      [92 packages]
 ✔ Uploaded image       [localhost:8228]

NAME                 VERSION                     TYPE
adduser              3.118ubuntu2                deb
apt                  2.0.2ubuntu0.2              deb
base-files           11ubuntu5.2                 deb
base-passwd          3.5.47                      deb
bash                 5.0-6ubuntu1.1              deb
bsdutils             1:2.34-0.1ubuntu9.1         deb
bzip2                1.0.8-2                     deb
coreutils            8.30-3ubuntu2               deb
dash                 0.5.10.2-6                  deb
debconf              1.5.73                      deb
debianutils          4.9.1                       deb
diffutils            1:3.7-3                     deb
dpkg                 1.19.7ubuntu3               deb
e2fsprogs            1.45.5-2ubuntu1             deb
fdisk                2.34-0.1ubuntu9.1           deb
findutils            4.7.0-1ubuntu1              deb
gcc-10-base          10.2.0-5ubuntu1~20.04       deb
gpgv                 2.2.19-3ubuntu2             deb
grep                 3.4-1                       deb
gzip                 1.10-0ubuntu4               deb
hostname             3.23                        deb
init-system-helpers  1.57                        deb
libacl1              2.2.53-6                    deb
libapt-pkg6.0        2.0.2ubuntu0.2              deb
libattr1             1:2.4.48-5                  deb
libaudit-common      1:2.8.5-2ubuntu6            deb
libaudit1            1:2.8.5-2ubuntu6            deb
libblkid1            2.34-0.1ubuntu9.1           deb
libbz2-1.0           1.0.8-2                     deb
libc-bin             2.31-0ubuntu9.1             deb
libc6                2.31-0ubuntu9.1             deb
libcap-ng0           0.7.9-2.1build1             deb
libcom-err2          1.45.5-2ubuntu1             deb
libcrypt1            1:4.4.10-10ubuntu4          deb
libdb5.3             5.3.28+dfsg1-0.6ubuntu2     deb
libdebconfclient0    0.251ubuntu1                deb
libext2fs2           1.45.5-2ubuntu1             deb
libfdisk1            2.34-0.1ubuntu9.1           deb
libffi7              3.3-4                       deb
libgcc-s1            10.2.0-5ubuntu1~20.04       deb
libgcrypt20          1.8.5-5ubuntu1              deb
libgmp10             2:6.2.0+dfsg-4              deb
libgnutls30          3.6.13-2ubuntu1.3           deb
libgpg-error0        1.37-1                      deb
libhogweed5          3.5.1+really3.5.1-2         deb
libidn2-0            2.2.0-2                     deb
liblz4-1             1.9.2-2                     deb
liblzma5             5.2.4-1ubuntu1              deb
libmount1            2.34-0.1ubuntu9.1           deb
libncurses6          6.2-0ubuntu2                deb
libncursesw6         6.2-0ubuntu2                deb
libnettle7           3.5.1+really3.5.1-2         deb
libp11-kit0          0.23.20-1ubuntu0.1          deb
libpam-modules       1.3.1-5ubuntu4.1            deb
libpam-modules-bin   1.3.1-5ubuntu4.1            deb
libpam-runtime       1.3.1-5ubuntu4.1            deb
libpam0g             1.3.1-5ubuntu4.1            deb
libpcre2-8-0         10.34-7                     deb
libpcre3             2:8.39-12build1             deb
libprocps8           2:3.3.16-1ubuntu2           deb
libseccomp2          2.4.3-1ubuntu3.20.04.3      deb
libselinux1          3.0-1build2                 deb
libsemanage-common   3.0-1build2                 deb
libsemanage1         3.0-1build2                 deb
libsepol1            3.0-1                       deb
libsmartcols1        2.34-0.1ubuntu9.1           deb
libss2               1.45.5-2ubuntu1             deb
libstdc++6           10.2.0-5ubuntu1~20.04       deb
libsystemd0          245.4-4ubuntu3.4            deb
libtasn1-6           4.16.0-2                    deb
libtinfo6            6.2-0ubuntu2                deb
libudev1             245.4-4ubuntu3.4            deb
libunistring2        0.9.10-2                    deb
libuuid1             2.34-0.1ubuntu9.1           deb
libzstd1             1.4.4+dfsg-3                deb
login                1:4.8.1-1ubuntu5.20.04      deb
logsave              1.45.5-2ubuntu1             deb
lsb-base             11.1.0ubuntu2               deb
mawk                 1.3.4.20200120-2            deb
mount                2.34-0.1ubuntu9.1           deb
ncurses-base         6.2-0ubuntu2                deb
ncurses-bin          6.2-0ubuntu2                deb
passwd               1:4.8.1-1ubuntu5.20.04      deb
perl-base            5.30.0-9ubuntu0.2           deb
procps               2:3.3.16-1ubuntu2           deb
sed                  4.7-1                       deb
sensible-utils       0.0.12+nmu1                 deb
sysvinit-utils       2.96-2.1ubuntu1             deb
tar                  1.30+dfsg-7ubuntu0.20.04.1  deb
ubuntu-keyring       2020.02.11.2                deb
util-linux           2.34-0.1ubuntu9.1           deb
zlib1g               1:1.2.11.dfsg-2ubuntu1.2    deb
❯ anchore-cli --u admin --p foobar image get ubuntu:latest
Image Digest: sha256:5146935f9248826d44dfc2489abfd5f4bdfbc319a738c04dfe1ef071f228a1ac
Parent Digest: sha256:5146935f9248826d44dfc2489abfd5f4bdfbc319a738c04dfe1ef071f228a1ac
Analysis Status: analyzed
Image Type: docker
Analyzed At: 2021-02-01T20:54:48Z
Image ID: sha256:f63181f19b2fe819156dcb068b3b5bc036820bec7014c5f77277cfa341d4cb5e
Dockerfile Mode: Guessed
Distro: ubuntu
Distro Version: 20.04
Size: 75289600
Architecture: amd64
Layer Count: 3
Annotations: anchore.system/import_operation_id=0decca824a6c480998d7fab5c0e40c4f

Full Tag: docker.io/ubuntu:latest
Tag Detected At: 2021-02-01T20:54:46Z



~/3.0_test on ☁️ zach@anchore.com(us-west1)
❯ anchore-cli --u admin --p foobar image vuln ubuntu:latest all
...
```

### Next Steps

Install [Syft](http://github.com/anchore/syft) to scan local images and generate software Bill-of-Materials to upload into your Anchore deployment.

After uploading the analysis, you'll need to use the CLI or UI to view vulnerabilities or policy evaluations using the enterprise feed data and policy features
such as [base-image diffs]({{< ref "/docs/overview/concepts/images/base_images" >}}) or [false positive management]({{< ref "/docs/overview/concepts/images/false_positive_management" >}}) 
