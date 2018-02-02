Title: VisualSVN pre-commit hook
Date: 2011-06-03 14:33
Author: Surya
Category: Rant
Slug: visualsvn-pre-commit-hook
Status: published

1.Start VisualSVN Server Manager  
2.Under Repositories, Select your repository and then Open Properties  
3.Select Hooks tab  
4.double click the pre-commit hook

Edit: As of VisualSVN version 2.5 use below code

    :::bash
    setlocal enabledelayedexpansion
    
    set REPOS=%1
    set TXN=%2
    
    set SVNLOOK="%VISUALSVN_SERVER%\bin\svnlook.exe"
    
    SET M=
    
    REM Concatenate all the lines in the commit message
    FOR /F "usebackq delims==" %%g IN (`%SVNLOOK% log -t %TXN% %REPOS%`) DO SET M=!M!%%g
    
    REM Make sure M is defined
    SET M=0%M%
    
    REM Here the 6 is the length we require
    IF NOT "%M:~6,1%"=="" goto NORMAL_EXIT
    
    :ERROR_TOO_SHORT
    echo "Commit note must be at least 6 letters" >&2
    goto ERROR_EXIT
    
    :ERROR_EXIT
    exit /b 1
    
    REM All checks passed, so allow the commit.
    :NORMAL_EXIT
    exit 0

If using VisualSVN \< 2.5

    :::bash
    setlocal
    
    set REPOS=%1
    set TXN=%2
    
    set SVNLOOK="C:\Program Files\VisualSVN Server\bin\svnlook.exe"
    
    REM Make sure that the log message contains some text.
    FOR /F "usebackq delims==" %%g IN (`%SVNLOOK% log -t %TXN% %REPOS% FINDSTR /R /C:......`) DO goto NORMAL_EXIT
    
    :ERROR_TOO_SHORT
    echo "Comment must be at least 6 letters" >&2
    goto ERROR_EXIT
    
    :ERROR_EXIT
    exit /b 1
    
    REM All checks passed, so allow the commit.
    :NORMAL_EXIT
    exit 0

The above code will make the comment of at least 6 letters mandatory.
