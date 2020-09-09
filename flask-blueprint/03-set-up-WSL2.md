## Setting Up WSL2 on Windows 10
Setting up WSL2 on Windows is a relatively painless process. 

As per Design Consideration #002 (don't recreate if you can leverage someone else's work), I'll defer documentation details to the mighty Microsoft communications team. You can find the latest steps and gotchas in their [official installation guide](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

With that said, I will note a few caveats that I encountered during my own upgrade effort:
* When I upgraded to WSL2 (mid-2020), joining the Windows Insider Program was mandatory to access the necessary build version (I was following [this post](https://char.gd/blog/2019/windows-web-dev-with-wsl2)). I believe Microsoft recently changed this requirement, because its official guide does not mention this as a precursor step.

* While updating to Windows version 2004, my update process froze at 61%. This kicked off a multihour troubleshooting session, ultimately isolating the problem as a known Conexant sound driver conflict. The problem was resolved by deleting everything in the `C:\Windows\SoftwareDistribution` folder, deleting the Conexant driver, and not reinstalling it until my system update was complete.

* Some guides advise installing the new [Windows Terminal](https://www.microsoft.com/en-ca/p/windows-terminal/9n0dx20hk701) after completing the WSL2 install. I struggled (and failed) to configure the fonts & colours to an acceptable state, ultimately deciding to run my WSL2 in the native Windows launcher (which looks and works just fine). 
<br>I likely am paying a performance price for having multiple WSL2 consoles open at once, but it hasn't seemed to impact me yet. YMMV. 

    

Previous: [Project Goals and Design Considerations](./02-project-goals-and-design-considerations.md)<br>
Next: [Install and Configure Docker Desktop](./04-install-and-configure-docker.md)
