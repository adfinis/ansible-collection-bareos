# Updating the roles

Roles in collection are managed as [git submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules).
* Changes can be fetched with `git submodule update --remote`.
* To pull changes for specific role, run: `git submodule update --remote <role name>` (Example: `git submodule update --remote roles/bareos_repository`).
