# Active Directory GPOs

## Goal

-   Create an Active Directory GPOs

-   Apply GPOs to member workstations

## Lab Setup

1.  In GNS3, open previous Active Directory Introduction project.

2.  Use the MMC Console saved in previous lab. Create the following (see
Pic for reference):

    Under "lab.com" create an OU: "Ducky Labs"

    Under "Ducky Labs" create OUs: Users, Computers and Groups

    Under "Ducky Labs\\Computers" create OUs: Sales and Marketing
\
\
![][1]

3.  From lab.com\\Users OU move users: Sally Smith and Bob Jones into lab.com\\Ducky Labs\\Users OU

4.  From lab.com\\Users move Groups: Sales and Marketing into lab.com\\Ducky Labs\\Groups OU

5.  From lab.com\\Computers move Computer: WIN10-VM-ITL into lab.com\\Ducky Labs\\Computers\\Sales

## GPO Creation

6.  In MMC, expand "Group Policy Management" snap-in. Expand it until "lab.com\\Ducky Labs\\Computers\\Sales" is viewable. The same list of OUs created earlier should be listed, if not restart the MMC. The tool may not pick up the changes without restarting it.

    In the GPO view only OUs are shown, no leaf objects. Right-Click on the "Sales" OU and select *Create a GPO in this domain and Link it here...*. Call the new GPO "Computer - Remove Settings and Control
Panel".

    **Note:** That the default "lab.com\\Computers" OU does NOT show in the GPO editor as a choice for attaching policies to.

7.  Right-Click on the new GPO and select Edit. This will open the GPO editor (note the "Standard" view option in the right-side pane is often easier to deal with as shown in image).
\
\
![][2]

8.  This image shows a hierarchy with two top categories Computer Configuration and User Configuration. In order to make this as sane as possible when building a GPO only work with the Computer Configuration side **OR** the User Configuration side. GPOs that need both do occur but are rare!

    Computer Configuration GPOs change options a specific computer NO MATTER WHO logs into it. These GPOs are assigned to OUs the contain computer AD objects. In our case "lab.com\\Ducky Labs\\Computers\\Sales".

    User Configuration GPOs change options that follow a user NO MATTER WHICH workstation they use. These GPOs are assigned to OUs the contain user AD objects. In our case "lab.com\\Ducky Labs\\Users".

    It is important to remember that GPOs cannot be assigned to a AD group only to OUs. This might mean that there is an OU with only one group in it just so a GPO can be assigned. (If the previous two sentences make your head hurt you are beginning to understand why GPOs are so complex).

9.  In the GPO editor open Computer Configuration -> Policies -> Administrative Templates: Policy definitions... -> Control Panel
\
\
![][3]

10. Double click on Settings Page Visibility to open the configuration window for that option. As shown on the right side there is some limited help for configuring choices regarding this GPO control. Sometimes they are helpful.... Uhhh sometimes not, Google is your friend here.
\
\
![][4]

11. Select the "Enabled" option. This will un-gray options below. Reading the right-side pane about the GPO control provides information about options. In this case we will set the Settings page on affected PCs to only show the "About" information of the PC. This will hide all the other settings options from the users. To do this in the "Settings Page Visibility:" fill in *showonly:about* and press the OK button to save.
\
\
![][5]

12. Once you have created and linked a GPO make sure your workstation is in the OU that the GPO is applied to. There are three options to get a GPO to apply.

    **First**, wait... GPOs are applied every 30 to 120 minutes \*yawn\*

    **Second**, in a command window run `gpupdate /force` command at a command prompt on the workstation. This is the recommended option. It might also need a logout and back in depending on the GPO involved.

    **Third**, reboot the workstation. Use this option when in doubt.

13. Login to Win10 as jonesb, open the start menu and select Settings. If the GPO is working when ANY user on the Win10 box goes to the settings page, they will only see the About information of the PC.
All other settings options are removed and inaccessible.
\
\
![][6]

14. Logout as jonesb and login as a-ducky and check the settings. What do you see?

## Trouble Shooting Tools

15. Log into the PC as jonesb. When many GPOs are in play it can be difficult to "see" what is going on. Admins often need to see the results of multiple GPOs applied to a PC and User combo. There are several ways to do this. The easiest is to go to the PC/User and have them run an RSoP report (Resultant Set of Policies).
\
If jonesb is having a problem, then we will need that user to run the report. Open a command window and run: `gpresult /r` to generate a report about the user's environment and GPOs. This will output to
the screen.
\
Copy/paste the screen output into to a notepad.exe file for later reference. Save that file to the desktop with the name jonesb.

16. To generate an HTML report, in a command windows run the command: `gpresult /H Desktop/jonesb-report.html` The screen output and HTML output are VERY different, and each can be helpful in different ways.

17. Using a web browser email both files to yourself for offline compairison.

18. Logout of the PC and then login as a-ducky and repeat both gpresult commands. Use filenames that have *a-ducky* and email them to yourself.

19. To see details about what a specific GPO does select the GPO in the MMC. The right pane will fill in with details about where it is being applied. Select the "Settings" tab and click on the "show all" button (if needed). Scroll down through Computer Configuration -> Policies -> Administrative Templates -> Control Panel -> Settings Page Visibility.

## GPO Inheritance

One of the most complex issues with GPOs is the idea of inheritance. Attaching a GPO to higher OU (closer to the top/root of the tree) will cause the GPO to be applied to any OUs farther down that branch (brains start melting here). This can cause... unexpected results.

20. Move the computer object WIN10-VM-ITL from the Sales OU to the Marketing OU. Reboot WIN10-VM-ITL and login as jonesb. Check Settings, can you see MORE than just the About info?

    Moving the Computer object to a different OU in the tree different GPOs apply to it. The previous GPO was attached to the Sales OU, so moving the object out of the "line of inheritance" of that GPO
removed its application.

    If the GPO were attached at the labs.com/Ducky Labs/Computers OU the move would not have changed the situation as that GPO would have then inherited to BOTH Marketing and Sales.


21. To disable application of a GPO without removing if from the hierarchy, in the MMC right-click on the GPO. Note the "Link Enable" option is checked. Uncheck it to have the GPO entry stay in the tree
but not be applied. This feature is VERY useful when trouble shooting.

22. To view GPO inheritance, click on the Sales OU, in the left pane select "Group Policy Inheritance". This shows not only inheritance, but also the ORDER (called Precedence) in which GPOs will be applied. GPOs are applied starting with the lowest numbers. The GPO with highest precedence (top of the list) will be fully applied as it's applied last.

    For example, a single OU has two GPOs attached.
        GPO-Red sets the desktop background to red
        GPO-Blue sets the desktop background to blue.

    **What color is the background when the user logs in?** That will depend on the precedence.


23. It is possible to block inheritance to part of the tree. It's VERY unusual to need this feature, but it does happen. Right-Click on an OU, in the context menu is the option "Block Inheritance". This will stop *normal* GPO inheritance from OUs above from applying.

24. With any rule there's an exception (sigh). Sometimes there is a need for a single GPO to inherit even down a tree with inheritance blocked! Right-Click on a GPO in the context menu is the option
"Enforced". This option will apply the GPO even through blocked inheritance options.

## GPO Exercises

Create separate GPOs for each exercise with descriptive names. Before attempting any of these exercises makes sure to disable (see above) any previous GPOs that were created. Interactions between GPOs can be very difficult to trouble shoot especially when just beginning with the
tool.

    To learn more about GPOs - **Google is your friend!**

### Simple

1.  Create an AD group and add smiths as a member. Write and attach a GPO that will put this group into the LOCAL administrators group of your workstation. Thus, making smiths an admin on the machine.

2.  Create a policy that removes the "lock option" on a workstation.

3.  Create a policy that forces the workstation to accept Remote Desktop Connection (RDC) connections (off by default).

### Moderate

4.  Create a policy that forces the Windows Firewall on but still allows users to create firewall exceptions.

5.  Create a policy that forces the user to use a specified non-default Windows Update settings.

6.  Block inheritance on an OU and learn what the effect is. Use the "enforced" setting on a GPO. Example use: Ohio University wants to push out a setting that would set screen savers to a specific
setting.

7.  Create a GPO that places an AD Group of users in the LOCAL machine Guest group. This should have the effect that those users will not have persistent user profiles on that machine.

### Difficult

8.  Create a GPO to have a .bat or .cmd script run at user login. Make it a different script for different users. These are called Login scripts.

9.  Create a GPO to deploy a small software package to your workstation. Hint: Only .MSI files can be used in a GPO software deployment.

10.  Create a "no-rights user". Create GPO(s) that lock that user down no matter what machine they login to. Set it up so that if the application is on a specified list the app won't run. (i.e., if
notepad.exe is the only app on the list they won't be able to run any other applications).

## Deliverables

1.  The first set of GPO reports were run as jonesb (a user with very little rights) you may have noticed something odd about both reports. Compare the output of the reports and describe why they are
different?

[1]: images/pic1.png
[2]: images/pic2.png
[3]: images/pic3.png
[4]: images/pic4.png
[5]: images/pic5.png
[6]: images/pic6.png