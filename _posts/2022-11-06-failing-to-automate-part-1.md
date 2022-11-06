---
title: Failing to automate - GitLab merge requests 
date: 2022-11-06 12:00:00
categories: [Automation] 
tags: [gitlab, automation, bash, git]
image:
  path: assets/2022-11/gitlab.png
  width: 1000
  height: 400
  alt: GitLab logo
--- 
Hello friend, after a long time, I am back with a new blog post. This time, I am getting sidetracked again and am writing about something different. I am going to talk about Gitlab and how I automated the little annoyance of creating merge requests, and finally how I truly learned two of the most common jokes in our industry are deeply rooted in truth. 

First of all, Git is amazing. When I initially started learning about Git, it felt more like a chore I have to do. After I started using it, I realized how amazing it is. It is the closest thing to a time machine we developers get (*redux coughs*). I started using Gitlab in my company, and I am lovin' it. It was a very smooth transition from Github to Gitlab. I am not going to talk about the differences between the two, but I will say that I am very happy with Gitlab.

The problem I had was not GitLab specifically, far from it actually. Following the trunk based development model, I create a new branch for every new feature I am working on. And sometimes, I have to create a new branch for every small change I am making. So, I have a lot of branches. And I have to create merge requests all of time. Every time I want to push my changes to remote, I have to first change my branch to `dev`, pull from remote, change my branch to the branch I am working on, and merge to the `dev` branch, and finally push to remote. Then, I have to go to Gitlab, create a merge request, select the source and target branch, and assign the right person. Luckly, if you authenticate GitLab using SSH, it provides a link to directly create a merge request. But again, you have to select the target branch, and assign the right person manually. It is a mild amount of annoyance, but if you have to do that 5 times a day, it adds up. 

At some point a few weeks back, I finally broke. I missed a few steps here and there, and it was causing issues either during merge, or worse, in staging environments. I decided to automate the process. I wanted to write a script that does all the song and dance of merging locally and creating a merge request with the right assignee and to the right branch.    

I started looking at the link GitLab shot out onto my terminal to look for hints on how to do just that. The link, once url decoded, basically looks like this `https://gitlab.com/0xecho/private_repo/merge_requests/new?merge_request[source_branch]=notMain`. What may directly catch you eye may be the `merge_request[source_branch]=notMain`. This is the branch you would currently want to merge. I went to the GitLab url, poped open DevTools and clicked Create Merge Request. I looked at the network tab, and Voila! I found the request that GitLab sends to create a merge request. It looks like this:

```json
{
    "merge_request": {
        "source_branch": "notMain",
        "target_branch": "dev",
        "title": "Merge notMain to dev",
        "assignee_ids": [
            123456
        ]
    }
}
```

Great Success! We now know what the data being sent to the server looks like, we can test the obvious hypothesis to make here: CAN WE SET THE `target_branch` AND `assignee_ids` TO WHATEVER WE WANT? The answer is unsurprisingly, yes. We can set both of them to whatever we want to the initial GET request and everything works as expected. In my excitement, I wrote a script that would create a merge request with the right assignee and to the right branch. 

```bash
#!/bin/bash

SOURCE_BRANCH=$(git rev-parse --abbrev-ref HEAD)
TARGET_BRANCH="dev"
ASSIGNEE_ID="123456"

echo "To create a merge request from $SOURCE_BRANCH to $TARGET_BRANCH, go to:"
echo "https://gitlab.com/0xecho/private_repo/merge_requests/new?merge_request[source_branch]="$SOURCE_BRANCH"&merge_request[target_branch]="$TARGET_BRANCH"&merge_request[assignee_ids][]="$ASSIGNEE_ID
```

I was hardcoding all the values, but atleast I was able to shave of that switching branches and assigning people manually. Hurrah! Little did I know, that was just the beginning of the rabbithole.

The next thing I wanted to automate was the switch, pull, switch, merge. As long I my working tree was clean, I can switch to any branch I want, pull and return to the branch I came from with no fear of having a merge conflict or uncommitted changes. I modified the script so that would do just that. 

```bash
#!/bin/bash

SOURCE_BRANCH=$(git rev-parse --abbrev-ref HEAD)
TARGET_BRANCH="dev"
ASSIGNEE_ID="123456"

git checkout $TARGET_BRANCH
git pull
git checkout $SOURCE_BRANCH
git merge $TARGET_BRANCH

echo "To create a merge request from $SOURCE_BRANCH to $TARGET_BRANCH, go to:"
echo "https://gitlab.com/0xecho/private_repo/merge_requests/new?merge_request[source_branch]="$SOURCE_BRANCH"&merge_request[target_branch]="$TARGET_BRANCH"&merge_request[assignee_ids][]="$ASSIGNEE_ID
```

It was working great, it got me wanting to create merge requests for the sake of it. Then after a bit I wanted more, not just because hardcoding there values was giving me nightmares from ghosts of engineers past, but because I needed to change the assignee for one MR and I had to find out manually what the assignee ID was. I know 🙈 so primitive. I wanted to automate that as well. I created a script that would fetch the assigneees and save it to a file, and also another file that will store the selected assignee.

```bash
#!/bin/bash

GITLAB_API_TOKEN="1234567890"

curl -s https://gitlab.com/api/v4/projects/0xecho%2Fprivate_repo/members/all -H "PRIVATE-TOKEN: $GITLAB_API_TOKEN" | jq -r '.[] | .name + " " + .id' > assigneees.txt
```

```bash
#!/bin/bash

echo "Enter the number of the assignee you want to select"
cat assigneees.txt | nl 

read -p "Enter the number: " assignee_number

assignee_id=$(cat assigneees.txt | sed -n "$assignee_number p" | awk '{print $2}')

echo $assignee_id | tee assignee_id.txt | xargs -I {} echo "Assignee ID set to {}"

```
So we can now select the assignee from a list of all the assigneees. I modified the script to read the assignee ID from the file, and the TARGET_BRANCH as an required argument.

```bash
#!/bin/bash

SOURCE_BRANCH=$(git rev-parse --abbrev-ref HEAD)

if [ -z "$1" ]
then
    echo "Please provide the target branch as an argument"
    exit 1
fi

TARGET_BRANCH=$1

ASSIGNEE_ID=$(cat assignee_id.txt)

if [ -z "$ASSIGNEE_ID" ]
then
    echo "Please set the assignee"
    exit 1
fi

git checkout $TARGET_BRANCH
git pull
git checkout $SOURCE_BRANCH
git merge $TARGET_BRANCH

echo "To create a merge request from $SOURCE_BRANCH to $TARGET_BRANCH, go to

git checkout $TARGET_BRANCH
git pull
git checkout $SOURCE_BRANCH
git merge $TARGET_BRANCH

echo "To create a merge request from $SOURCE_BRANCH to $TARGET_BRANCH, go to:"
echo "https://gitlab.com/0xecho/private_repo/merge_requests/new?merge_request[source_branch]="$SOURCE_BRANCH"&merge_request[target_branch]="$TARGET_BRANCH"&merge_request[assignee_ids][]="$ASSIGNEE_ID
```

I was happy with how this little script turned out. But then I scrolled down on the GitLab docs page I was refering to. I saw this: `You can use Git push options to perform certain actions for merge requests at the same time as pushing changes:`. It the goes on to explain how I could have done `git push -o merge_request.create -o merge_request.target=dev -o merge_request.assignee_id=123456` and it would have created the merge request for me. I was upset, I had spent so much time on this, and I could have done it in a fraction of the time. But then I realized, I had learned a quite few of things along the way, and I had created the switch, pull, switch, merge part that I still use.

Programmers really do try to automate a task that can be done manually in 15 seconds, and then spend 15 hours trying to do it. Also, not reading the docs for 5 more seconds can save you from creating a "solution" with more flaws than a 5 year old's drawing of a unicorn.

I know this was more of a rambling than a post, but I wanted to share my experience with you guys. I hope you enjoyed reading this as much as I enjoyed writing it. I would love to hear your thoughts on this.  

Thank you for reading! 🙏 Until next week! 🤓 Goodbye friend! 👋