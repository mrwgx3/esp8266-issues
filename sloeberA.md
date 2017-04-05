### Basic Infos

#### Hardware

```
Hardware:     ESP12, Adafruit Huzzah Breakout and Feather Boards  
Core Version: 2.3.0
Computer:     Dell, Intel Core i7 CPU, 870 @ 2.93Ghz,  16.0 GB RAM
```

#### OS, IDE

``` 
OS:   Windows 10 Professional and Linux Ubuntu 16.04LTS   
IDE:  V4.0_win64.2017-01-17_15-14-48 (stable release)
      io.sloeber.product-4.0.1-20170401.212426-33-win32.win32.x86_64 (nightly build)
```

### Description

After downloading and doing the default install of V4 for Windows 10, I found the 'spaces-in-path' issue subsequently described in ['Install Advice' under topic 'Windows Comments'][spaces_in_path]:
```
'spaces Windows users keep in mind that spaces cause problems. So do not install the arduino IDE eclipse or Sloeber in
"program files". Don't create the workspace in "my documents".'
```
For those of us which jump first and ask questions later, it would be nice if you could change the default workspace path in future builds such that they don't contain spaces (the workspace shown for my install was 'C:\Users\DELL\My Documents\sloeber-workspace')

Thanks for a job well done!

[spaces_in_path]: http://eclipse.baeyens.it/installAdvice.shtml

### Requested Install Information
```
  C/C++ Autotools support	9.2.1.201703062208	org.eclipse.cdt.autotools.feature.group	Eclipse CDT
  C/C++ Common GDB Support	9.2.1.201703062208	org.eclipse.cdt.gdb.feature.group	Eclipse CDT
  C/C++ Development Platform	9.2.1.201703062208	org.eclipse.cdt.platform.feature.group	Eclipse CDT
  C/C++ Development Tools	9.2.1.201703062208	org.eclipse.cdt.feature.group	Eclipse CDT
  C/C++ DSF GDB Debugger Integration	9.2.1.201703062208	org.eclipse.cdt.gnu.dsf.feature.group	Eclipse CDT
  C/C++ GNU Toolchain Build Support	9.2.1.201703062208	org.eclipse.cdt.gnu.build.feature.group	Eclipse CDT
  C/C++ GNU Toolchain Debug Support	9.2.1.201703062208	org.eclipse.cdt.gnu.debug.feature.group	Eclipse CDT
  ECF Core Feature	1.3.0.v20160823-2221	org.eclipse.ecf.core.feature.feature.group	Eclipse.org - ECF
  ECF Core SSL Feature	1.1.0.v20160823-2221	org.eclipse.ecf.core.ssl.feature.feature.group	Eclipse.org - ECF
  ECF Filetransfer Feature	3.13.2.v20160823-2221	org.eclipse.ecf.filetransfer.feature.feature.group	Eclipse.org
  ECF Filetransfer SSL Feature	1.1.0.v20160823-2221	org.eclipse.ecf.filetransfer.ssl.feature.feature.group	Eclipse.org - ECF
  ECF Httpclient4 Filetransfer Provider	3.13.2.v20160823-2221	org.eclipse.ecf.filetransfer.httpclient4.feature.feature.group	Eclipse.org - ECF
  ECF Httpclient4 Filetransfer SSL Provider	1.1.0.v20160823-2221	org.eclipse.ecf.filetransfer.httpclient4.ssl.feature.feature.group	Eclipse.org - ECF
  Eclipse 4 Rich Client Platform	1.5.3.v20170228-0512	org.eclipse.e4.rcp.feature.group	Eclipse.org
  Eclipse Help System	2.2.2.v20170301-0400	org.eclipse.help.feature.group	Eclipse.org
  Eclipse Platform	4.6.3.v20170301-0400	org.eclipse.platform.feature.group	Eclipse.org
  Eclipse RCP	4.6.3.v20170301-0400	org.eclipse.rcp.feature.group	Eclipse.org
  EMF - Eclipse Modeling Framework Core Runtime	2.12.0.v20160420-0247	org.eclipse.emf.ecore.feature.group	Eclipse Modeling Project
  EMF Common	2.12.0.v20160420-0247	org.eclipse.emf.common.feature.group	Eclipse Modeling Project
  Equinox p2, backward compatibility support	1.2.203.v20170131-1444	org.eclipse.equinox.p2.extras.feature.feature.group	Eclipse.org - Equinox
  Equinox p2, Discovery UI support	1.0.401.v20160901-1335	org.eclipse.equinox.p2.discovery.feature.feature.group	Eclipse.org - Equinox
  Equinox p2, headless functionalities	1.3.203.v20170131-1444	org.eclipse.equinox.p2.core.feature.feature.group	Eclipse.org - Equinox
  Equinox p2, minimal support for RCP applications	1.2.203.v20170131-1444	org.eclipse.equinox.p2.rcp.feature.feature.group	Eclipse.org - Equinox
  Equinox p2, Provisioning for IDEs.	2.2.203.v20170131-1444	org.eclipse.equinox.p2.user.ui.feature.group	Eclipse.org - Equinox
  Git integration for Eclipse	4.6.1.201703071140-r	org.eclipse.egit.feature.group	Eclipse EGit
  Java implementation of Git	4.6.1.201703071140-r	org.eclipse.jgit.feature.group	Eclipse JGit
  Marketplace Client	1.5.4.v20170222-1941	org.eclipse.epp.mpc.feature.group	Eclipse Marketplace Client
  Mylyn Commons	3.21.0.v20160707-1856	org.eclipse.mylyn.commons.feature.group	Eclipse Mylyn
  Mylyn Commons Connector: Discovery	3.21.0.v20160729-1739	org.eclipse.mylyn.discovery.feature.group	Eclipse Mylyn
  Mylyn Commons Connector: Monitor	3.21.0.v20160630-1702	org.eclipse.mylyn.monitor.feature.group	Eclipse Mylyn
  Mylyn Commons Identity	1.13.0.v20160630-1702	org.eclipse.mylyn.commons.identity.feature.group	Eclipse Mylyn
  Mylyn Commons Notifications	1.13.0.v20160721-2347	org.eclipse.mylyn.commons.notifications.feature.group	Eclipse Mylyn
  Mylyn Commons Repositories	1.13.0.v20160630-1702	org.eclipse.mylyn.commons.repositories.feature.group	Eclipse Mylyn
  Mylyn Context Connector: Eclipse IDE	3.21.0.v20160912-1820	org.eclipse.mylyn.ide_feature.feature.group	Eclipse Mylyn
  Mylyn Context Connector: Team Support	3.21.0.v20160701-1337	org.eclipse.mylyn.team_feature.feature.group	Eclipse Mylyn
  Mylyn Task List	3.21.0.v20160914-0252	org.eclipse.mylyn_feature.feature.group	Eclipse Mylyn
  Mylyn Task-Focused Interface	3.21.0.v20160815-2336	org.eclipse.mylyn.context_feature.feature.group	Eclipse Mylyn
  Mylyn Tasks Connector: Bugzilla	3.21.0.v20160909-1813	org.eclipse.mylyn.bugzilla_feature.feature.group	Eclipse Mylyn
  Mylyn WikiText	2.10.1.v20161129-1925	org.eclipse.mylyn.wikitext_feature.feature.group	Eclipse Mylyn
  Nebula Oscilloscope Widget	1.2.0.201703302228	org.eclipse.nebula.widgets.oscilloscope.feature.feature.group	Eclipse Nebula
  Remote System Explorer End-User Runtime	3.7.2.201610260947	org.eclipse.rse.feature.group	Eclipse TM Project
  Sloeber	4.0.1.201704012122	io.sloeber.feature.feature.group	Jan Baeyens and Others
  Sloeber, the Eclipse Arduino IDE	4.0.0.201704012122	io.sloeber.product	null
```
