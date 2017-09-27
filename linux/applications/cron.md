## Editing Cron

1. `crontab –e`
2. MIN HOUR DOM MON DOW CMD
```
MIN   Minute field  0 to 59
Hour  Hour field    0 to 23
DOM   Day of Month  1 to 31
MON   Month field   1 to 12
DOW   Day of Week   0 to 6
CMD   Any Command
```
3. Restart Cron

## Cron under different users

`crontab -u <user> -e`
