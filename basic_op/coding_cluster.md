# **3. Coding Workflow and Cluster Computing**
---  
#### 3.1 Git and GitLab  
In our lab, we use Git and GitLab to manage code and collaborate efficiently.<br>

**What are Git and GitLab?**
- Git is a version control system installed on your computer. It tracks changes to files and lets you revert to earlier versions if needed.
- GitLab is a free online platform (similar to OneDrive or Google Drive, but designed for code) where your Git-tracked files are stored, shared, and collaborated on.
- Think of Git as your personal notebook for code, and GitLab as the shared library where everyone contributes and reviews each other’s work.<br>


**How do they work together?**
- Git synchronizes specific folders (called repositories) on your laptop with GitLab. When you make changes locally, you can push them to GitLab. When others make changes, you can pull them to your computer.<br>


**Why do we use it (and why you should too)?**
- Undo Mistakes: Easily roll back to previous versions of your code.
- Synchronization: Keep your code consistent across laptops, lab computers, and servers.
- Backup: Your work is safely stored and versioned online.
- Collaboration: Team members can review, suggest edits, and merge changes into shared projects.


**Getting started**<br>
To set it up, you need to:
1. Install Git
2. Create a GitLab account
3. Link Git and GitLab using an SSH key


*1. Install Git*  <br>
Git may already be installed on your laptop. If not, download and install it:
- Linux: Use your package manager (`sudo apt install git`, etc.)  
- Mac: [https://git-scm.com/download/mac](https://git-scm.com/download/mac)  
- Windows: [https://git-scm.com/download/win](https://git-scm.com/download/win) 
<br>

*2. Create a GitLab account* 
- Sign up at [https://git.sbg.ac.at](https://git.sbg.ac.at) to create a PLUS gitlab instance, create an account on [gitlab](https://about.gitlab.com/) or use an existing account.
- Log in.
<br>

*3. Link Git and GitLab using an SSH key*
- SSH stands for Secure Shell and allows secure access to GitLab without entering your password every time. Think of it as creating a digital 'key pair': a *private key* that stays on your computer and a *public key* that gets added to your GitLab account. These are used to authenticate you and to enable a communication between the `git` command and the GitLab server.

<div style="margin-left: 80px">

*Check if you already have keys*  
- Open a terminal and type:  `ls ~/.ssh`<br>  
- If you see the files `id_rsa`and `id_rsa.pub`, you already have a key. If you get an error, you have to create one.

*Create a new key (if needed)*
- Type `ssh-keygen`, accept the default location, and set a password if you want for extra security.
- When the command has finished, you will find the two files we were looking for previously in the `~/.ssh` folder. The one with the `.pub`extension is the public key. This i what GitLab needs.

*Add the key to GitLab*
- In the terminal, enter: `cat ~/.ssh/id_rsa.pub`. Then, go to [GitLab](https://gitlab.com/users/sign_in)/profile/keys and paste the output of the cat command in the field called `Key`and click on `Add Key`.
- Check whether it has worked by entering `ssh git@gitlab.com` into the terminal. It worked if you get an output like this:

<div style="margin-left: 60px">

```bash
PTY allocation request failed on channel 0
Welcome to GitLab, @xxxx!
Connection to gitlab.com closed.
```
</div>
</div>
<br>
Congratulations! You just successfully prepared your laptop and are ready to start a new analysis project! Read in [this](xxx) tutorial on how to do that.<br>
<br>

This part of the tutorial was adapted from [a tutorial written by Thomas Hartmann](https://thht.gitlab.io/tutorials/using_python_for_science/01_prepare_computer_for_scientific_python/git.html) - check it out if you'd like! Further resources provided by Git you may find helpful are the [Git official documentation](https://git-scm.com/doc) and the [GitLab manual](https://docs.gitlab.com). Additionally, [this Datacamp course](https://app.datacamp.com/learn/courses/introduction-to-git) and [this guide](https://rogerdudler.github.io/git-guide/) are helpful further tutorials.
<br>


#### 3.2 Cluster Computing 
For CPU-heavy code, the lab uses the cluster of the University, called SCC (= Salzburg Collaborative Computing). You don't need to install things on your local laptop for it, just know for now that it exists. <br>
Since cluster computing involves many steps, we wrote a separate [tutorial](xxx) that walks you through everything in detail. Check it out whenever you're ready to dive deeper! Additionally, there is a [tutorial](https://scc.plus.ac.at/pages/scc-pilot-documentation.html) on the official SCC website, which contains all important information regarding using the cluster.
<br>
<br>