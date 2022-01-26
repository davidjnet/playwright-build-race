# playwright-build-race

Reproduces a race condition where multiple projects share the same output (bin) directory and have a transitive dependency on Microsoft.Playwright.

When the projects rebuild or clean in parallel, they all try deleting the .playwright folder. They get errors where e.g. one project opens the directory
to enumerate it while another project is trying to delete it; e.g. one project has enumerated a directory but another project deletes the files before the
first project can do so.

`dotnet build -bl -t:Rebuild` errors quite reliably on my machine (Win10, NTFS, NVME disk, 24 cores) in this example with five projects building at once.

Example failure:
```C:\Users\David.James\.nuget\packages\microsoft.playwright\1.18.0\buildTransitive\Microsoft.Playwright.targets(33,5): error MSB3231: Unable to remove directory "C:\git\playwright-build-race\\common-bin\net6.0\\.playwright". Could not find a part of the path '\\?\C:\git\playwright-build-race\common-bin\net6.0\.playwright\package\lib\common'. [C:\git\playwright-build-race\proj-a\proj-a.csproj]
C:\Users\David.James\.nuget\packages\microsoft.playwright\1.18.0\buildTransitive\Microsoft.Playwright.targets(33,5): error MSB3231: Unable to remove directory "C:\git\playwright-build-race\\common-bin\net6.0\\.playwright". Access to the path '\\?\C:\git\playwright-build-race\common-bin\net6.0\.playwright\package\lib\third_party\highlightjs' is denied. [C:\git\playwright-build-race\proj-e\proj-e.csproj]
C:\Users\David.James\.nuget\packages\microsoft.playwright\1.18.0\buildTransitive\Microsoft.Playwright.targets(33,5): error MSB3231: Unable to remove directory "C:\git\playwright-build-race\\common-bin\net6.0\\.playwright". Access to the path '\\?\C:\git\playwright-build-race\common-bin\net6.0\.playwright\package\lib\third_party\highlightjs\highlightjs' is denied. [C:\git\playwright-build-race\proj-d\proj-d.csproj]
C:\Users\David.James\.nuget\packages\microsoft.playwright\1.18.0\buildTransitive\Microsoft.Playwright.targets(33,5): error MSB3231: Unable to remove directory "C:\git\playwright-build-race\\common-bin\net6.0\\.playwright". Access to the path '\\?\C:\git\playwright-build-race\common-bin\net6.0\.playwright\package\lib\third_party\highlightjs\highlightjs' is denied. [C:\git\playwright-build-race\proj-b\proj-b.csproj]
C:\Users\David.James\.nuget\packages\microsoft.playwright\1.18.0\buildTransitive\Microsoft.Playwright.targets(33,5): error MSB3231: Unable to remove directory "C:\git\playwright-build-race\\common-bin\net6.0\\.playwright". Access to the path '\\?\C:\git\playwright-build-race\common-bin\net6.0\.playwright\package\lib\webpack' is denied. [C:\git\playwright-build-race\proj-c\proj-c.csproj]
```
