This blog post is in response to David Cowen's Weekly Challenge as seen in his Daily Blog #610.

The three questions I will be answering in this post include the following:
1. What within a ShellBags entry would tell you how the user had set their directory viewing preferences?
2. What is the default view if no settings are changed?
3. If a user attempts to access a directory where they are not permitted, what directory viewing settings are left behind?
***
What are these Shellbags we speak of? There are a lot of great resources published about an in-depth analysis of Shellbags, which made some introductory research of this topic easy to jump into. Shellbags are an artifact where the actions of the user are tracked when elements such as folders and ZIP files are accessed, renamed, clicked, cut, copied, or even deleted. This allows a forensicator to understand what folders or places a suspected user has accessed. After investigating further, these questions will pursue the other information that can be found within Shellbags. Since Windows tracks almost everything, it should be no surprise that there is some content about Shellbags that can be expanded on.

For my explanation, I’ve decided to use Windows 10 1709 (Build 16299.15) in a virtual machine. The system is freshly installed. Tools on this machine include FTK Imager, Procmon, Eric Zimmerman’s ShellbagsExplorer and RegistryExplorer.

So, onto those questions!

# 1. What within a ShellBags entry would tell you how the user had set their directory viewing preferences?

To start breaking down this question, I will first explain a little background information about the ways in which Shellbags are stored. There are quite a few locations that need to be checked and cross-referenced to get the answer to these questions.

Since I’m using Windows 10, the primary locations where I will be focusing are the following:
    %UserProfile%\NTUSER.dat
    %UserProfile%\AppData\Local\Microsoft\Windows\UsrClass.dat
