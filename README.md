django-ldap3-sync
================
django-ldap3-sync is a fork of [django-ldap-sync]("https://github.com/jbittel/django-ldap-sync", "django-ldap-sync") originally created by Jason Bittel.
django-ldap3-sync introduces the following features:
	- Uses the [ldap3 library]("https://github.com/cannatag/ldap3", "ldap3") for ldap communication. ldap3 is pure python and python 3 compatible.
	- Can synchronize group membership directly out of the ldap directory.
	- Can manage deletion of groups / users in the directory by suspending or deleting those objects in Django.
	- Will update existing Django users groups if information changes in the directory.
	- Paged LDAP searches
	- LDAP Server Pools

django-ldap3-sync provides a Django management command that synchronizes LDAP
users and groups from an authoritative server. It performs a one-way
synchronization that creates and/or updates the local Django users and groups.

This synchronization is performed each time the management command is run and
can be fired manually on demand, via an automatic cron script or as a periodic
`Celery`_ task.

Quickstart
----------

#. Install the application::

      pip install django-ldap3-sync

#. Append it to the installed apps::

      INSTALLED_APPS = (
          # ...
          'ldap3_sync',
      )

#. Configure the required `settings`_.

#. Run the synchronization management command::

      manage.py syncldap

For more information on installation and configuration, see the included
documentation or read the documentation online at
`django-ldap3-sync.readthedocs.org`_.

Configuration
-------------

### User Synchronization Options

**LDAP_SYNC_USER_FILTER** -- Required

Default: ""

The filter used to retrieve users from the directory. Must be in standard LDAP filter syntax as per [RFC2254]("http://www.ietf.org/rfc/rfc2254.txt?number=2254", "RFC 22545")

**LDAP_SYNC_USER_BASE**

Default: The value of LDAP_SYNC_BASE

The distinguished name of the container to base the search for users in.

**LDAP_SYNC_USER_ATTRIBUTES** -- Required

Default: ""

A dictionary of key value pairs where the keys are the names of ldap fields and the values are the names of corresponding django model fields. New users will be created with these fields populated and existing users will have these fields updated.

**LDAP_SYNC_USER_EXEMPT_FROM_SYNC**

Default: []

A list of usernames corresponding to Django users who should be excluded from the sync. Useful for Administrative users who do not have a corresponding user in the directory.

**LDAP_SYNC_USER_REMOVAL_ACTION**

Default: NOTHING

The action to take when a user no longer exists in the directory. Possible values are NOTHING, SUSPEND and DELETE. Note that the SUSPEND option uses the Django user models is_active field and sets it to false.


**LDAP_SYNC_USER_SET_UNUSABLE_PASSWORD**

Default: True

If true this uses the django method set_unusable_password on all newly created users. Useful where django authentication will not be used.

**LDAP_SYNC_USERS**

Default: True

Controls whether users should be synchronized from the directory.

###Group Synchronization Options

**LDAP_SYNC_GROUP_FILTER

Default: ""

The filter used to retrieve users from the directory. Must be in standard LDAP filter syntax as per [RFC2254]("http://www.ietf.org/rfc/rfc2254.txt?number=2254", "RFC 22545")

**LDAP_SYNC_GROUP_BASE

Default: The value of LDAP_SYNC_BASE

The distinguished name of the container to base the search for groups in.

**LDAP_SYNC_GROUP_ATTRIBUTES

Default: ""

A dictionary of key value pairs where the keys are the names of ldap fields and the values are the names of corresponding django model fields. New groups will be created with these fields populated and existing users will have these fields updated.

**LDAP_SYNC_GROUP_REMOVAL_ACTION

Default: NOTHING

The action to take when a group no longer exists in the directory. Possible values are NOTHING and DELETE.

**LDAP_SYNC_GROUP_EXEMPT_FROM_REMOVAL

Default: []

A list of group names of Django groups that should be excluded from the sync.

**LDAP_SYNC_GROUPS

Default: True

Controls wether groups should be synchronized.

###Membership Synchronization Options

**LDAP_SYNC_GROUP_MEMBERSHIP

Default: True

Controls wether groups will be synchronized from the directory.

**LDAP_SYNC_GROUP_MEMBERSHIP_FILTER

Default: (&(objectClass=group)(member={user_dn}))

The filter used to retrieve the groups that a user belongs to. {user_dn} will be replaced with the distinguished name of the user.

###LDAP Server Options

**LDAP_CONFIG -- Required

Default: None

Configuration item used to configure the server pool. LDAP_CONFIG can contain the following keys:
	
	**page_size
	Default: 500
	The page size for searches using this server pool.

	**bind_user
	Default: None
	The distinguished name of the user to bind to the directory with.

	**bind_pass
	Default: None
	The password of the user to bind to the directory with.

	**pooling_strategy
	Default: ROUND_ROBIN
	The strategy to use when the pool contains multiple servers. See ldap3 documentation at https://ldap3.readthedocs.org/en/latest/servers.html#server-pool for more information. Can be FIRST, ROUND_ROBIN or RANDOM.

	**servers
	Default: None
	A list of dictionaries each one containing configuration information for a server. Possible server configuration keys are:

		**address
		Default: None
		Either the IP address or FQDN of the directory server.

		**use_ssl
		Default: False
		Use SSL with this conntection.

		**timeout
		Default: 30
		Connection timeout with this server.

		**get_schema
		Default: SCHEMA
		Determines which schema information to retrieve from the server. At a minimum this should be SCHEMA so that values retrieved from the directory are coerced to proper python types. 


.. _Celery: http://www.celeryproject.org
.. _settings: http://django-ldap3-sync.readthedocs.org/en/latest/settings.html
.. _django-ldap3-sync.readthedocs.org: http://django-ldap3-sync.readthedocs.org