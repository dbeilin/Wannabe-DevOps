# Direnv

If you’re bouncing between multiple AWS accounts or credentials in your CLI, this tool might be a huge quality of life perk for you.

Basically, direnv can load and unload environment variables depending on the current directory you’re in. In this post I’ll show how to use it to switch between AWS credentials.

Let’s start with a real world example right of the bat:
```bash
~/playground master* ❯ ls -l
total 20
drwxr-xr-x 2 dbei dbei 4096 Apr 21 09:37 project1
drwxr-xr-x 2 dbei dbei 4096 Apr 21 09:37 project2
~/playground master* 
```

* Project1 will use AWS Account #1
* Project2 will use AWS Account #2

**Let’s start with Project1:**

1. Setup [direnv](https://direnv.net/).

2. cd into your desired directory. In my case it would be `project1`.
   
3. Create **.envrc**: `vi .envrc`
   
4. Add your AWS credentials which you can generate in IAM:
   ```
   export AWS_ACCESS_KEY_ID=AKIAZXIDSFDSNTLBJWC
   export AWS_DEFAULT_REGION=eu-west-1
   export AWS_SECRET_ACCESS_KEY=KJuja33rTxIlK/wee/DsdrdhjKKg/HDrmtreDJ
   ```
5. `cd` in and out of that directory.


6. You should see an error: 

    ???+ error
        ```
        direnv: error /playground/project1/.envrc is blocked. Run direnv allow to approve its content
        ```

7. Type `direnv allow` and you should be good:
```bash
~/playground/project1 master* ❯ direnv allow
direnv: loading ~/playground/project1/.envrc
direnv: export ~AWS_DEFAULT_REGION
```

8. You can test that it works with any AWS CLI command. For example:
```bash
~/playground/project1 master* ❯ aws s3 ls
2022-03-22 16:35:07 daniel-awesome-bucket
2022-04-21 09:18:52 do-not-delete-gatedgarden-audit-384466123559
~/playground/project1 master* ❯
```

Repeat the same process for Project2 and use different credentials. After that, you should be set. From here you can move between folders, which will automatically move between your AWS credentials and accounts!

???+ tip

      Don’t forget to add `.envrc` to your `.gitignore`!
