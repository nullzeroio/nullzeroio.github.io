---
title: Locked Out of vCenter 5.1
date: 2013-09-20T15:38:39+00:00
author: Kevin Kirkpatrick
layout: post
image: /wp-content/uploads/2013/09/VPX_ACCESS_1.jpg
categories:
  - vSphere 5.x
tags:
  - Locked out of vCenter
  - SQL
  - SSO
  - vCenter Database
  - vSphere 5.1
---
Recently, I was called in to help a client out with a vCenter 5.1 install and came across the somewhat common issue of being locked out of vCenter (which is most common after the upgrade process). After some investigation, it appeared the proper Identity Sources were configured and SSO, in general, looked okay. After scratching our heads a bit, I decided to take a look inside the vCenter DB and verify account/group access. Since this was a clean installation, and not an upgrade, the only account in the vCenter DB was the one specified during the installation wizard. This was a SQL DB, so the table where this access can be found is in the &#8220;VMW.VPX_ACCESS&#8221; table, within the vCenter DB.

Note: If you are going to attempt this procedure, make sure you have a good/valid backup of the entire DB that you can restore.

To verify/modify access:

1. Stop all vCenter Services

2. Use SQL Management Studio to connect to the DB

3. Expand the vCenter DB (in my case, the name is &#8216;VCDB&#8217;)

4. Expand the Tables and right click on the &#8220;VMW.VPX_ACCESS&#8221; table; select &#8220;Edit Top 200 Rows&#8221;.

5. You should see a single row (if this is a new install) with the group/account details that you setup as part of the install wizard, in the &#8220;PRINCIPAL&#8221; column.

[<img src="http://www.vmotioned.com/wp-content/uploads/2013/09/VPX_ACCESS_1.jpg" alt="VPX_ACCESS_1" width="543" height="69" class="aligncenter size-full wp-image-429" srcset="http://www.vmotioned.com/wp-content/uploads/2013/09/VPX_ACCESS_1.jpg 543w, http://www.vmotioned.com/wp-content/uploads/2013/09/VPX_ACCESS_1-300x38.jpg 300w" sizes="(max-width: 543px) 100vw, 543px" />](http://www.vmotioned.com/wp-content/uploads/2013/09/VPX_ACCESS_1.jpg)

6. Make any necessary changes to the account details and close the table

7. Restart vCenter services and see if access has been restored

In this particular scenario, it was found that the client entered the incorrect details during the install wizard, which is why no one was able to access vCenter.