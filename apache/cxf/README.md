Web Browsing of SVN

To browse via the web use the ViewVC interface:

http://svn.apache.org/viewvc/cxf/trunk/

Or via Fisheye courtesy of Atlassian:

http://fisheye6.atlassian.com/browse/cxf

Or to browse the source tree directly:

http://svn.apache.org/repos/asf/cxf/trunk/

Checking out from SVN

The source code can be checked out anonymously over HTTP by doing:
 

svn co http://svn.apache.org/repos/asf/cxf/trunk cxf
 
Committers can check out the code over HTTPS:
 

svn co https://svn.apache.org/repos/asf/cxf/trunk cxf
 
Committers needing to check out a specific branch can do so as follows:
 

svn co https://svn.apache.org/repos/asf/cxf/branches/{branch name} cxf
 
Setting up your subversion client

When adding files to subversion, it's important that your subversion client is properly setup to the appropriate subversion properties are set. The client can do it automatically by modifying the auto-props section of the subversion config file. Use the contents of:
 

http://svn.apache.org/repos/asf/cxf/trunk/etc/svn-auto-props
 
Using git

Apache also maintains a git mirror of the svn repository. You can clone the git repository via:
 

git clone git://git.apache.org/cxf.git
 
or
 

git clone http://git.apache.org/cxf.git
 
For committers/developers interested in using git for CXF trunk development, Apache has some documentation:
