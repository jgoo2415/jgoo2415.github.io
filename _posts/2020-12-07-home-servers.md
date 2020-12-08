---
title: "Importance of a Home Media Server"
categories: 
  - Git
---

# Overview

A home media server is a very valuable tool for anybody that writes software because you can store your own Git repositories.
It's possible to just use some existing cloud-based repository solution like Github and while that may be easier, that doesn't mean it's the best option available.
There are a number of factors which could push a developer towards making their own storage solution for code and related projects.


# Reasons to Migrate

I think there are a few different reasons why developers might wish to migrate to a home solution for their Git repositories:

1. Github is only free as long as your repositories are small. If you need to store a gigabyte of files (for whatever reason), then Github is not going to work out. This point is particularly relevant for video game developers because their projects can get pretty large.

2. Github may not be considered as a "trusted zone" because ultimately the repositories are stored outside of your home. Someone can potentially log into my Github account (however they may do so) and then do malicious things to my private repositories. However, we have full control over our home networks and what we want to expose to the outside world.
This sort of thing really only applies for developers that need Git repositories for their proprietary projects and they want peace of mind.

3. Home servers can be used for multiple purposes. Gitea is one home solution for Git repositories and it's very lightweight. There should be a lot of room for running other types of services on the server in tandem, like Jenkins maybe.

4. Home servers can also be used for storing build artifacts that you might not want uploaded to Github. A binary repository like Artifactory can be used to store a lot of things from Dockre images to Java jars and it has support for a lot of other things as well.

5. Github will straight up take down some repositories if they think that there's anything illegal going on (the point being is that they may not have evidence before the decision). Youtube-dl for example was taken down for a time because it was said to breach Youtube's terms, but that wasn't actually the case.
Due to this situation, the youtube-dl developers were in a situation where they might have lost their hard work and were blocked from continuing their development for a time.


# Considerations

A home server is nice for a number of reasons, but you need to put the time into making your data safe. Software or power failures can easily corrupt a file and give you a hard time, so backing up important files is a very good way to keep a history of file changes.
There's also the possibility for hard drives to fail altogether between backups, so it might be a good idea to setup a RAID of hard drives so that files are duplicated. Using both of these systems in tandem seems to be the best way to ensure that your data is never lost within your home network, but one or the other would probably be sufficient for most people.

If your main reason for a home server was to keep your repositories safe, then it might be a bad idea to expose your repository's port to the outside world. My home server is only accessible through LAN so I can't access it while traveling, but I know that somebody would have to somehow login to my router before being able to attempt to login to my Git repositories.


# Conclusion

I've been storing my programs at home for almost a year now and it has been a pretty great experience. I like having the control and being able to have huge repositories when needed. There's a lot of pros, but it takes a good amount of work and research to get setup the right way. I'd recommend it for those who are curious.