# Promote replica database to master

1. Setup a new database replication if necessary (for example: increase size or the existing one will be under maintenance)
2. Disable delayed job:   (sudo monit -g dj_msuper stop). **NOTE:  DO IT IN BOTH APP SERVERS**
3. Disable Cron tab  .  **NOTE: DO IT IN BOTH APP SERVERS**
    
    >crontab -l > my_cron_backup.txt (backup)

    >crontab -r (delete)
4. Backup database  
<https://support.cloud.engineyard.com/hc/en-us/articles/205408068-Back-Up-the-Database>
5. check the replica db is in sync with master by runing the below query, it should less than 60 seconds. 
**After turn on maintenance mode the sync will stopped.**

  > psql -U postgres -t -c "SELECT (now() - pg_last_xact_replay_timestamp()) AS time_lag;"
6. Disable App (turn on maintenance mode)
7. Promote the slave database to master  
<https://support.cloud.engineyard.com/hc/en-us/articles/205408228-Promote-a-Database-Replica>
  
8. Redeploy App(this should start delayed job and cron automatically)
    >crontab my_cron_backup.txt (restore crontab)

    >not need to run the below two, they should already done by deployment
          enable crontab
          enable delayed job   (sudo monit -g dj_msuper start)
9. Enable App(turn off maintenance mode)

---

10. Create another replication database
11. Remove old dbs cos they are not used but still running and chargable

    Terminate the old replica db
    Terminate the old master db (because it is detached)

---

A certain amount of downtime is unavoidable, however our standard promotion process usually only takes a few minutes. Our documentation regarding this process is available at https://support.cloud.engineyard.com/hc/en-us/articles/205408228-Promote-a-Database-Replica, however here's the short version:

1. Put up your maintenance page
2. stop any background workers or cron jobs that talk to the database.
3. Click the "Promote" button, which will conduct some checks, promote the DB replica and then run chef to update the environment.
4. Redeploy the app
5. Take down the maintenance page.

If you have long-running custom chef recipes, the "Apply" stage can sometimes cause this to take longer than you'd want. Likewise, if your deploys usually take a long time then this can affect how long it takes to restart services.

The purpose of the redeploy is really just to ensure that all services are restarted with the updated config. If you prefer, you can manually restart services that talk to the database - with planning, this is faster than a redeploy.

Additionally, the point of putting up the maintenance page is to prevent the app from serving up errors when the DB is unavailable. If you are careful, you can have your app gracefully degrade when DB access is unavailable (for example: avoid unnecessary DB activity, serve cached content, automatically display friendly error messages on DB connection failure). You can also have background workers automatically sleep or resubmit jobs for a later attempt when they encounter a DB connection failure. If the app can do all of this automatically, then a maintenance page becomes unnecessary. At that point, the downtime is limited to the amount of time between the promotion completing and services being restarted.

I hope this helps - please let us know if you have any questions, or if there's anything else we can do to assist!

Yours,

