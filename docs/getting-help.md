# Getting Help
If you have any issues or questions regarding Muscadine, please contact Oliver Higginbotham via odh49@msstate.edu or submit questions to the #helpdesk-muscadine thread channel in our SIG-HPC Discord.

If you have any issues with accounting or login, submit a helpdesk ticket to help@hpc.msstate.edu

---

## FAQ

Q: What if I don't want to build a program from source?
: A: Womp womp.

Q: **HELP!!!** I Accidentally delete a file. what can I do?
: A: If it was on `/scratch`: Womp Womp. If it was on `/home` or `/work` and was greater than a day old, you may find it in it's corresponding path in `/home/.snapshots` or `/work/.snapshots` respectively

Q: How do I run across the whole cluster?
: A: Generally, students are permitted to run across all 4 compute nodes by using the windfall QoS. Note that the windfall qos is preemptible, meaning jobs running against the normal qos take priority and will cause windfall jobs to suspend should they need to.
: Should a student need to use the 5th node (for hero runs, etc), a hidden partition can be made available temporarily upon request/justification.

Q: Why can't I download files w/ curl or wget?
: A: Only the head node allows downloads from the internet, you're probably trying to download something on a compute node.

```{warning}
Do not wait to retrieve deleted files. snapshots are automatiaclly purged after 30 days
```
