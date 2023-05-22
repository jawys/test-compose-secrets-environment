1. Copy the following `compose.yaml`
   ``` yaml
   version: "3.8"

   secrets:
     api_pass:
       environment: SOME_API_PASS

   services:
     backend:
       image: node:alpine
       environment:
         - API_PASS_FILE=/run/secrets/api_pass
       secrets:
         - source: api_pass
       command:
         - sh
         - -c
         - set -x; ls -la /run/secrets; id
       user: node

   name: test-compose-secrets-environment
   ```
2. Run **docker compose up** with `SOME_API_PASS` env set inline
   ``` shell
   ❯ SOME_API_PASS=SOME_API_PASS docker compose -f compose.yaml up
   [+] Running 1/2
   ✔ Network test-compose-secrets-environment_default      Created              0.0s
   ⠋ Container test-compose-secrets-environment-backend-1  Creating             0.0s
   Error response from daemon: getent unable to find entry "node" in passwd database
   ```
3. Seeing an error?! Try again...
   ``` shell
   ❯ SOME_API_PASS=SOME_API_PASS docker compose -f compose.yaml up
   Attaching to test-compose-secrets-environment-backend-1
   test-compose-secrets-environment-backend-1  | + ls -la /run/secrets
   test-compose-secrets-environment-backend-1  | ls: /run/secrets: No such file or directory
   test-compose-secrets-environment-backend-1  | uid=1000(node) gid=1000(node) groups=1000(node)
   test-compose-secrets-environment-backend-1  | + id
   test-compose-secrets-environment-backend-1 exited with code 0
   ```
4. Comment out `user: node` directive so that user is root
   ``` diff
   --- a/compose.yaml
   +++ b/compose.yaml
   @@ -15,6 +15,6 @@ services:
         - sh
         - -c
         - set -x; ls -la /run/secrets; id
   -    user: node
   +    # user: node

   name: test-compose-secrets-environment
   ```
5. Run compose up again running the container as root
   ``` shell
   ❯ SOME_API_PASS=SOME_API_PASS docker compose -f compose.yaml up
   [+] Running 1/1
   ✔ Container test-compose-secrets-environment-backend-1  Recreated            0.1s
   Attaching to test-compose-secrets-environment-backend-1
   test-compose-secrets-environment-backend-1  | + ls -la /run/secrets
   test-compose-secrets-environment-backend-1  | /run/secrets:
   test-compose-secrets-environment-backend-1  | total 12
   test-compose-secrets-environment-backend-1  | drwxr-xr-x    2 root     root          4096 May 22 13:36 .
   test-compose-secrets-environment-backend-1  | drwxr-xr-x    1 root     root          4096 May 22 13:36 ..
   test-compose-secrets-environment-backend-1  | -r--------    1 root     root            13 May 22 13:36 api_pass
   test-compose-secrets-environment-backend-1  | + id
   test-compose-secrets-environment-backend-1  | uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
   test-compose-secrets-environment-backend-1 exited with code 0
   ```
