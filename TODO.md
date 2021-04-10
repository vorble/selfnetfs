# TODO

## Next Version Features

* Better path sanity checks (no `:`).
* Sanity checks for user names in useradd and usermod. Also password.
* Think about adding a feature where an admin user can connect as another user easily
  (from a Session, make a new Session as the other user).
* For the server, have the program take an environment variable, configuration
  option, or argument to specify the port number to listen on.

## Monorepo Refactoring

* [ ] Look at `package.json` `repository` field and see if you can give it a directory within the repository.
* [ ] Figure out if I want to generate map files in the tsconfig for each project. (I lean toward Yes)
* [ ] Deep review each package.json file for consistency.
* [ ] Figure out how lerna manages propagating package version numbers through the project. Does it even?
* [ ] Write basic test that starts a server and runs some requests against it.
* [ ] Revise naming all through the project.