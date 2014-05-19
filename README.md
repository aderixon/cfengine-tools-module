# Cfengine module:tools

module:tools is a Cfengine 2.x module that generates a Cfengine policy file
called cf.tools for updating software packages compiled and installed from
source. It is based on the software depot scheme described in _The Practice of
System and Network Administration_ by Thomas A. Limoncelli and Christine Hogan.

The module takes a colon-separated list of package directories as an
argument (to work around argument limits in Cfengine), and generates a
Cfengine rules file that can be imported into your existing policy. The
rules specify how to update the package files from a master server and
create the necessary symlinks. There are also rules to simply check that
the listed packages are installed and generate warnings if any are not
found. Define the Check and/or Update classes to decide what actions are
taken.

## Use

The Cfengine rules required to incorporate this module and the resulting
rule file are:

    control:
        AllowRedefinitionOf = ( tools )
        workdir = ( /var/cfengine )
        moduledirectory = ( $(workdir)/modules )
        pkginstall = ( $(workdir)/inputs/cf.pkgsinstall )
        depot = ( /tools )    # location for all s/w
        mainserver = ( fileserver )    # master software server
    classes:
        PolicyServer = ( name of Cfengine master policy host )
        Hr00::
            Check = ( any )    # check packages on all hosts daily
        Sunday.Hr00::
            Update = ( hosts to update weekly )
        !PolicyServer::
            pkgcfexists = ( IsPlain($\{pkginstall\}) )    # exclude policy server
    import:
        (Check|Update).pkgcfexists::
            $(pkginstall)
    control:
        # define package lists for each group of hosts
        # (example)
        any::
            tools = ( "openssh-4.0p1:openssl-0.9.7e:grep-2.4.2" )    # common
        internal::
            tools = ( "$(tools):samba-3.4" )
        development::
            tools = ( "$(tools):gcc-3.4:make-3.79" )
        # create toolsinstall rules
        (Check|Update).!PolicyServer::
            actionsequence = ( "module:tools $(tools)" )

module:tools relies on [Graft](http://freshmeat.net/projects/graft/) or a
similar symbolic link managing utility.

For more information, see [my blog](http://www.big-bubbles.fluff.org/blogs/bubbles/blog/2005/08/05/automated-software-update-with-cfengine/)

## Support

This is legacy code made available for reference purposes only.
