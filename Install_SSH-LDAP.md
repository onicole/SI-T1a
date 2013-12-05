Add SSH and LDAP compatibility
----------------------------------------

1. Installing SSSD  
Start by installing SSSD. Use *yum* like this.  
`yum install sssd`

2. Edit */etc/nsswitch.conf*  
	Now, you have to modify the *nsswitch.conf* file. Add "sss" at the end of these lines :
	 * passwd
	 * shadow
	 * group
	 * services
	 * netgroup

	It should be something like this
	
		passwd:     files sss
		shadow:     files sss
		group:      files sss
		
		services:   files sss
		
		netgroup:   files sss

3. Copy the *sssd.conf* file from this dir to **/etc/sssd/**

4. Modify the permissions of *sssd.conf* to **600**  
`chmod 600 sssd.conf`

5. Create a group named *ldap_users*  
`groupadd ldap_users`

6. Write down the gid of this new group  
`cat /etc/group | grep ldap_users`

7. Edit the *sssd.conf* file :
 * L.27 | override\_gid = *put the ldap_users's gid here*
 * L.32 | ldap\_default\_bind_dn = *put a cn of a user who can bind the ldap*
 * L.34 | ldap\_default\_authtok = *put the password of this user*

	Example :
	
		override_gid = 500
		ldap_default_bind_dn = cn=MERMOD Allan,ou=SI-T1a,ou=Informatique,ou=Eleves,dc=sc,dc=cpnv,dc=ch
		ldap_default_authtok_type = password
		ldap_default_authtok = MyPassword1234

8. Execute this command line :  
`authconfig --enablesssd --enablesssdauth --update`

9. And now, just restart sssd like this :  
`service sssd restart`

10. DONE ! :)