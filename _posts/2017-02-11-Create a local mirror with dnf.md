---
layout: post
title: Create a local mirror with dnf
category: Linux
---

With the introduction of fedora 25, I need to install the nvidia driver with negativo17's repo. But sadly it doesn't have any mirror around me so the connection always fail and the download speed is what we call a catastrophe. Creating a local repo should be helpful and doing it with dnf is mostly the same with yum. But the procedure is quite straightforward.


<a id="org9661a7d"></a>

# Step 0:

Select or create a directory where you want all the rpm packages to be stored, somewhere like `~/Others/repos/`


<a id="org9da0d27"></a>

# Step 1:

Sync all the packages from the remote repo you like to the local directory you just created with dnf reposync command. I created a little script for this, the while loop here is to deal with connection failure(which prove my bad network connection), remember to place the script file in the target directory before you run it:

```bash
#!/bin/bash

dnf reposync --repoid repository-id
while [ $? -eq 1 ]
do
    dnf reposync --repoid repository-id
done
exit
```

When the syncing is done, keep the working directory in the directory which contains all the packages you just downloaded, then use createrepo<sub>c</sub> command to create the rpm-md format repository:

```bash
createrepo_c --database ./
```

the **&#x2013;database** option would create a sqlite database to accelerate the dnf command when you need to install packages from the local repo.

Another way to do the same thing is via `wget`, you can use the following long command to download and update the rpms:

```sh
#!/bin/sh
wget -m -np \
     -R xml,html,gz,bz2,*tmp \
     -A rpm,drpm \
     -e robots=off \
     -c -r https://url-to-repo/arch
```

After `wget` complete, do `createrepo_c` as above mentioned.


<a id="orgd95fd76"></a>

# Step 2:

Create a .repo file in `/etc/yum.repos.d/`, the .repo file should looks like this:

```conf
[id-for-the-repo]
name = name of the repo
baseurl = file:///path/to/repo
```

Just replace the id and name with whatever you want. Then dnf should recognize the new local repository.

To this point, you can disable the online repo permanently via:

```bash
dnf config-manager --set-disabled [repo]
```

or temporarily when installing packages:

```bash
dnf --disablerepo [repo]
```

or add the following line:

```conf
fastertmirror=True
```

to `/etc/dnf/dnf.conf` in main section, which will tell dnf to find the fastest mirror automatically. The detailed description about dnf.conf can be found in man page by command:

```bash
man dnf.conf
```


<a id="org67fc5e2"></a>

# Update the repo:

```bash
createrepo_c --update --database ./
```

I have noticed that all the online documentation about creating local mirror still use yum up to this post being written. So if you have never used yum before, this post should be helpful.
