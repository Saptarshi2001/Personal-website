---
title: "Building a Shell from Scratch"
subtitle: "Systems Programming"
---

Hello and welcome readers!! So this is the first post on my blog. And to kick things off, let's see how we can build a basic implementation of a shell. If you have ever wondered how a shell actually works, you can get to know it here.

## Reading the Lines

First, we declare a buffer size (let's say 1024; it can be anything really but just big enough to hold the input), a variable called `line` that points to the array, a variable called `position` that indicates the current index, and an integer variable `c`.

We then use `malloc` to dynamically allocate the array. We dynamically allocate it since we can't predefine a definite number of characters from the start.

Next, we enter a loop and input characters through `c = getchar()`. If it's EOF or '\n', we end the loop, put `line[position]` in the array, end the loop, and return the pointer to the array. Otherwise, we store that character in the array and increase the position by 1. If the size of the array exceeds `buffsize`, we use `realloc` to reallocate the array.

```c
char *readline()
{
    int status;
    char *line;
    int buff_size = LSH_RL_BUFSIZE;
    int position = 0;
    int c;
    line = (char *)malloc(buff_size * sizeof(char));
    if (!line)
    {
        printf("No space allocated");
        return NULL;
    }

    while(1)
    {
        c=getchar();
        if(c=='\n'||c==EOF)
        {
            line[position]='\0';
            break;
        }
        else if(position>=buff_size)
        {
            buff_size+=LSH_RL_BUFSIZE;
            line=realloc(line,sizeof(char)*buff_size);
            if(!line)
            {
                printf("%s","Reallocation unsuccesful");
            }
        }
        else
        {
            line[position]=c;
            position++;
        }
    }
    line[position]='\0';
    return line;
}
```

## Splitting the Line

After reading the line, we pass the pointer to a function `splitline`. Here we declare a variable `buff_size` to be 64. We declare `ptrtokens` and `token` as a pointer and another as the pointer to pointer variable. Next, we dynamically allocate the `tokens` variable as an array where each element will be a variable.

We then go through a loop where using the `strtok` function, we split the line array using delimiters, get the pointer that points to these split lines, and put it in the array of pointers. We increment `pos` by 1 and continue until `token != NULL`. We then return the pointer that points to the array of pointers, i.e., `tokens`.

Before executing it, we create an array of built-ins for the commands that will be executed. We also create a function that stores the addresses of the functions of the commands we execute.

```c
char **splitline(char *line)
{
    int buff_size = LSH_TOK_BUFSIZE;
    char **tokens;
    char *token;
    int pos = 0;
    tokens = malloc(sizeof(line) * buff_size);
    if (!tokens)
    {
        fprintf(stderr, "Allocation unsuccesfull");
        return NULL;
    }
    token = strtok(line, LSH_TOK_DELIM);
    while (token != NULL)
    {
        tokens[pos] = token;
        pos++;
        if (pos >= buff_size)
        {
            buff_size += LSH_TOK_BUFSIZE;
            tokens = realloc(tokens, buff_size * sizeof(char *));
            if (!tokens)
            {
                fprintf(stderr, "Reallocation unsuccesfull");
                return NULL;
            }
        }
        token = strtok(NULL, LSH_TOK_DELIM);
    }
    tokens[pos] = NULL;
    return tokens;
}
```

## Executing the Commands

Now that we have the tokens, we pass them to `sh_execute`. Here, we check if the first token is one of the built-in commands. If it is, we call the corresponding function. Otherwise, we call `sh_launch` which uses `execvp` to execute the command.

## Changing Directory

The `cd` command is implemented as `sh_cd`. If no argument is given, it prints the current directory. If the argument is `..`, it moves up one directory. Otherwise, it uses `chdir` to change to the specified directory.

```c
int sh_cd(char **args)
{
    char* store;
    if (strcmp(args[0],"cd")==0 && args[1]==NULL)
    {
        store=_getcwd(store,sizeof(MAX_PATH));
        printf("%s ",store);
        return 1;
    }
    else if(strcmp(args[1],"..")==0)
    {
        chdir("..");
    }
    else
    {
        if (chdir(args[1])!=0)
        {
            fprintf(stderr, "Oops");
            printf("\n");
            return 0;
        }
    }
    store=_getcwd(store,sizeof(MAX_PATH));
    printf("%s ",store);
    return 1;
}
```

## Making a Directory

Next up, we go to `sh_mkdir`. This function helps us to create a directory. If there are no arguments, it gives an error. If the directory already exists or the path is not found, it throws an error. Otherwise, it uses the `CreateDirectory` function, which takes the argument and `NULL` to create the directory.

```c
int sh_mkdir(char **args)
{
    int result;
    result=CreateDirectory(args[1],NULL);
    if(args[1]==NULL)
    {
        fprintf(stderr,"No directory created \n");
        return 0;
    }
    else if(result==0)
    {
        if(errno==ERROR_ALREADY_EXISTS)
        {
            fprintf(stderr,"Directory already present \n");
            return 0;
        }
        else if(errno==ERROR_PATH_NOT_FOUND)
        {
            fprintf(stderr,"Path not found \n");
            return 0;
        }
    }
    printf("Directory created \n");
    char* store;
    store=_getcwd(store,sizeof(MAX_PATH));
    printf("%s ",store);
    return 1;
}
```

## Help, CLS, and Exit

For getting a list of all the commands, we iterate over all the built-ins and print them. For `exit` and `cls`, there's not much to explain.

```c
int sh_help()
{
    printf("\n");
    printf("HERE ARE THE COMMANDS \n");
    for (int j = 0; j < size(); j++)
    {
        printf("%s\n", builtins[j]);
        printf("\n");
    }
    char* store;
    store=_getcwd(store,sizeof(MAX_PATH));
    printf("%s ",store);
    return 1;
}

int sh_exit()
{
    return 0;
}
int sh_clear()
{
    system("cls");
    char* store;
    store=_getcwd(store,sizeof(MAX_PATH));
    printf("%s ",store);
    return 1;
}
```

## Changing the Color

This one is my favorite. We first print out the characters and numbers present. We then take one input for the background color and one for the foreground color. We join both of them with `snprintf` and use the `system` command. Let's say we use 1 as the background and E as the foreground, and we get the desired color scheme.

```c
int sh_color()
{
    char col;
    char backcol;
    printf("Press any of the keys in the table for seting background and foreground color \n");
    printf("Here's the table \n");
    printf("Color id\tColor\tColor id\tColor \n");
    printf( "1\t\tAqua\tA\t\tLight  Green \n");
    printf( "2\t\tGreen\t0\t\tBlack \n");
    printf( "3\t\tBlue\t9\t\tLight  Blue \n");
    printf( "4\t\tRed\tB\t\tLight  Aqua \n");
    printf( "5\t\tPurple\tC\t\tLight  Red \n");
    printf( "6\t\tYellow\tD\t\tLight  Purple \n");
    printf( "7\t\tWhite\tE\t\tLight  Yellow \n");
    printf( "8\t\tGray\tF\t\tBright White \n");
    printf("Enter the background color:- ");
    scanf(" %c",&backcol);
    printf("Enter the foreground color:- ");
    scanf(" %c",&col);
    char command[50];
    snprintf(command,sizeof(command),"Color %c%c",backcol,col);
    system(command);
    char* store;
    store=_getcwd(store,sizeof(MAX_PATH));
    printf("%s ",store);
    return 1;
}
```

## Extracting Contents from a File

Lastly, we see how the `cat` command can be used to extract contents from a file into the console. As of now, this command only works on text files, but I will add functionality for other types of files as well. If there's no `cat` in `args[0]` or no file given in `args[1]`, it's an error. Otherwise, we take `args[1]`, use the `fopen` function in C, and pass the file pointer to `fgets`. What `fgets` does in this loop is read the text line from the console, which we print out until it reaches `NULL`.

```c
int sh_cat(char **args)
{
    char *err="cat";
    char *contains=".txt";
    FILE *ptr;

    if(strcmp(args[0],"cat")!=0)
    {
        printf("%s","oops not a cat command");
        return 0;
    }
    char result[sizeof(args[1])];
    ptr=fopen(args[1],"r");
    if(ptr==NULL && strstr(args[1],contains))
    {
        ptr=fopen(args[1],"w");
    }
    else if(ptr==NULL)
    {
        printf("%s","Oops no file");
        return 0;
    }
    while (fgets(result,sizeof(LSH_RL_BUFSIZE),ptr)!=NULL)
    {
        printf("%s",result);
    }
    fclose(ptr);
    char *store;
    store=_getcwd(store,sizeof(MAX_PATH));
    printf("%s",store);
    return 1;
}
```

## Now Reading All of These

Now putting readline, splitline and execute into a single function we get:

```c
void read_all()
{
    char *line;
    char **split;
    int status;
    printf("Welcome to the holy command line \n");

    do
    {
        printf(">>");
        line = readline();
        split = splitline(line);
        status= sh_execute(split);
    } while (status);

    free(line);
    free(split);
}
```

## Putting it all together

```c
#define LSH_RL_BUFSIZE 1024
#define LSH_TOK_BUFSIZE 64
#define LSH_TOK_DELIM " \t\r\n\a"
#include <windows.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include<tchar.h>
#include<conio.h>
#include<unistd.h>
#include<assert.h>
#include<sys/stat.h>
#include<direct.h>
#include"shell.h"
#include"test.c"
const char *builtins[] = {"cd","mkdir","help", "exit","echo","cls","color","cat"};
int (*builtins_fun[])(char **) =
{
    &sh_cd,
    &sh_mkdir,
    &sh_help,
    &sh_exit,
    &sh_echo,
    &sh_clear,
    &sh_color,
    &sh_cat
};

int size()
{
    return sizeof(builtins) / sizeof(char *);
}

// ... (all the functions defined above)

int main()
{
    read_all();
}
```

## Conclusion

And that's the end of the blog. Here's the source [code](https://github.com/Saptarshi2001/Cshell). You can check out the roadmap for this project over [here](https://github.com/Saptarshi2001/Cshell?tab=readme-ov-file#roadmap). Until then, goodbye!
