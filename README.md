# deploy2020

The Studentenportal deployment currently being deployed on
new.studentenportal.ch (aka vshsr01.nine.ch). Soon to be the replacement for
studentenportal.ch.

To deploy, do the following:

- Make sure you can access the server via SSH using key-based authentication.
- Clone the "pass" repository so it's inside this repository under pass/
- Run "ansible-playbook site.yml"
