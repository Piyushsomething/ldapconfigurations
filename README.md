# ldapconfigurations
## Docker command

docker network create my-network --driver bridge

docker run --detach --rm --name openldap   --network my-network   --env LDAP_ADMIN_USERNAME=admin   --env LDAP_ADMIN_PASSWORD=adminpassword   --env LDAP_USERS=customuser   --env LDAP_PASSWORDS=custompassword   --env LDAP_ROOT=dc=example,dc=org   --env LDAP_ADMIN_DN=cn=admin,dc=example,dc=org -p 1389:1389  bitnami/openldap:latest


## python code
from ldap3 import Server, Connection, ALL, SUBTREE, MODIFY_REPLACE
from ldap3.core.exceptions import LDAPException, LDAPBindError

# LDAP server details
LDAP_SERVER = 'localhost'
LDAP_PORT = 1389  # Default non-privileged port for OpenLDAP
LDAP_ADMIN_DN = 'cn=admin,dc=example,dc=org'
LDAP_ADMIN_PASSWORD = 'adminpassword'
LDAP_BASE_DN = 'dc=example,dc=org'

def connect_and_bind(dn, password):
    try:
        server = Server(LDAP_SERVER, port=LDAP_PORT, use_ssl=False, get_info=ALL)
        conn = Connection(server, user=dn, password=password, auto_bind=True)
        print(f"Successfully bound as {dn}")
        return conn
    except LDAPBindError as e:
        print(f"Bind failed for {dn}: {e}")
    except LDAPException as e:
        print(f"LDAP error: {e}")
    return None

def main():
    # First, try to bind as admin
    conn = connect_and_bind(LDAP_ADMIN_DN, LDAP_ADMIN_PASSWORD)
    if not conn:
        print("Admin bind failed. Please check admin credentials.")
        return

    # Search for users
    search_filter = '(objectClass=person)'
    try:
        conn.search(LDAP_BASE_DN, search_filter, SUBTREE, attributes=['cn', 'uid'])
        if len(conn.entries) > 0:
            print("\nUsers found:")
            for entry in conn.entries:
                print(f"DN: {entry.entry_dn}, CN: {entry.cn}, UID: {entry.uid}")
        else:
            print("No users found")
    except LDAPException as e:
        print(f"Search error: {e}")

    # Always unbind when done
    conn.unbind()

if __name__ == "__main__":
    main()


## python code 2
import sys
from ldap3 import Server, Connection, ALL, SUBTREE
from ldap3.core.exceptions import LDAPException

# LDAP server details
LDAP_SERVER = 'localhost'  # Change this if your LDAP server is on a different host
LDAP_PORT = 1389  # Default non-privileged port for OpenLDAP
LDAP_USER = 'cn=admin,dc=example,dc=org'
LDAP_PASSWORD = 'adminpassword'
LDAP_BASE_DN = 'dc=example,dc=org'

def connect_ldap():
    try:
        # Define the server
        server = Server(LDAP_SERVER, port=LDAP_PORT, get_info=ALL)
        
        # Create the connection
        conn = Connection(server, user=LDAP_USER, password=LDAP_PASSWORD, auto_bind=True)
        
        print(f"Successfully connected to LDAP server at {LDAP_SERVER}:{LDAP_PORT}")
        return conn
    except LDAPException as e:
        print(f"Failed to connect to LDAP server: {e}")
        return None

def list_users(conn):
    try:
        # Search for all users
        conn.search(search_base=LDAP_BASE_DN,
                    search_filter='(objectClass=person)',
                    search_scope=SUBTREE,
                    attributes=['cn', 'uid'])
        
        print("\nList of users:")
        for entry in conn.entries:
            print(f"DN: {entry.entry_dn}")
            print(f"CN: {entry.cn}")
            print(f"UID: {entry.uid}")
            print("---")
        
        return True
    except LDAPException as e:
        print(f"Error while searching for users: {e}")
        return False

def main():
    conn = connect_ldap()
    if conn:
        if list_users(conn):
            print("Successfully listed users.")
        else:
            print("Failed to list users.")
        conn.unbind()
    else:
        print("Exiting due to connection failure.")
        sys.exit(1)

if __name__ == "__main__":
    main()
