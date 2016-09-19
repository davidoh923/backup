#!/bin/bash
# You need to pass an argument for either daily or archive so we know which bucket to put the backup. 

# Setup AWS CLI
# sudo apt-get install awscli
# Setup AWS config on /.aws/config

# Crontab
# 0 6 * * 1,2,3,4,5,6 /root/confluence_backup.sh daily
# 0 6 * * 0 /root/confluence_backup.sh archive

# Set the time stamp
DATE=$(date +"%Y%m%d%H%M")

#Back up the AWS RDS database
pg_dump -v -c -C -h RDS Endpoint -U username -f /path/to/sqlfile${DATE}.sql confluence

#Tar it up including Confluence program files, data directory, database sql file and Apache config
tar -czf /path/to/backupfile{DATE}.tar.gz /data/confluence/ /etc/apache2/sites-available/000-default.conf /opt/atlassian/confluence/ /etc/ssl/ /path/to/sqlfile${DATE}.sql

#Upload it to the appropriate S3 folder
aws s3 --profile default cp /path/to/backupfile{DATE}.tar.gz 
s3://path/to/${1}/backupfile${DATE}.tar.gz --region=us-west-1

#Get rid of the local file
rm -f /path/to/backupfile{DATE}.tar.gz /path/to/sqlfile${DATE}.sql
