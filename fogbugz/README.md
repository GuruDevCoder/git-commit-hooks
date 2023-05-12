###Why do we have this?###

In order to validate your commits a client commit hook must be installed on your repository. This is a Python script that it's executed each time you try to commit on your repo and it does these checks:

  * checks that the commit reference one or more fogbugz cases, looking for these formats anywhere in the commit message:
    * [123456]
    * case:123456
    * case#123456
  * checks that the cases referenced exist
  * checks that the cases are open

If any of those checks are failing the commit will be refused. You can use the special keyword NOFOGZ anywhere in your commit to skip the control, or you can use "-n" parameter in your commit to exclude all your local verification, but please do that only in case of emergencies (you may have to explain why)


###How do I install it?###

Steps:
*  make sure you have Python 2.5+ installed (*python --version*): if not please follow the instruction for your operating system to install it, Be aare that Python3 is not supported
*  install the [Fogbugz client library](https://developers.fogbugz.com/default.asp?W197) (if you have pip just digit *pip install fogbugz* or *sudo pip install fogbugz* if the previous do not work)
*  copy to your repository the commit-hook script getting it from the repo.Assuming you are sitting in the repository root folder you can execute this two commands:

```bash
curl https://raw.githubusercontent.com/workshare/git-commit-hooks/master/fogbugz/commit-msg -o .git/hooks/commit-msg
chmod +x .git/hooks/commit-msg
```


###How do I configure it?###

The script needs to access fogbugz and for that reason you will need to setup an access token before starting using the script the first time. Assuming you are sitting in the repository root folder you have to execute this command:


```bash
.git/hooks/commit-msg setup your-email@domain.com your-fogbugz-password
```


Note that you have to do this *only once* as the configuration is kept on a file on your local home (no need to do this step for every repository you setup the hook against)

 
###How does it work?###

Ok, imagine you have the hook loaded on your repository and you decide you want to commit something. In your commit message you have to reference the case this commit is related to: the simplest and cleanest thing you can do is prepend it to your commit message

```
$> git commit -m "[24587] This is my first commit"

> FOGBUGZ commit hook  === c/o Workshare Ltd ===
>
> Detected cases ['24587']
> Checking cases...
>  case 24587, OPEN - Unable to do upgrade the goffaw (via api) for restricted files
>
> SUCCESS - Commit accepted!

[master 981dcab] This is my first commit [24587]
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 444
```
When the criteria are not accepted, a meaningful message will be displayed and the commit will be rejected :)
Note that you can reference multiple cases in your commit message, mixing the reference syntax used, but then all of them will be tested and verified.

###Troubleshooting###

**Q. When I commit I am getting this message:**

```bash
FAILED - Temporarily unable to connect to fogbugz: Error Code 3: <![CDATA[Not logged in]]>
```

What should I do?

**A.** Your access token to fogbugz has been compromised or it's not valid anymore. Please re-execute the setup as described in the "How do I configure it?" section of this wiki page


**Q. When I commit I am getting this message:**

```bash
File ".git/hooks/commit-msg", line 119
    print '> '

      SyntaxError: invalid syntax
```
What should I do?

**A.** You are using Python3, where [print statements are now functions](http://stackoverflow.com/questions/826948/syntax-error-on-print-with-python-3) (hooray!). To sort this out you will have to install Python2 and setup the fogbugz library using the manual install procedure.


**Q. This commit hook does not with [Sourcetree](http://www.sourcetreeapp.com/)**

**A.** This is a common issue with this Atlassian product: if this is happening to you you will have to add a preliminary batch to fix the system path before running the real commit hook. So please rename the commit-msg file to commit-msg-internal and create a new commit-msg file (same place) with these contents:
```bash
export PATH="/usr/local/bin:$PATH"
$(dirname $0)/commit-msg-internal $*
```
