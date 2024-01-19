I automated a silly task as a beginner’s project in Python. It was a nice learning experience as I don’t do this quite often.

So we have a list of upcoming new employees in an Excel file. It has their name and start date. There are a few people in our office that need a reminder that new people are joining, so this script sends them that email with all the details.

Generally, if there’s a new employee on that list that starts in less than 10 days, it adds him to a table and sends that in an email. I run this script in a cron-job once a week at the same hour.

```py
import pandas as pd
import datetime as dt
import smtplib, ssl
from email.mime.multipart import MIMEMultipart
 
def sendEmail():
    sender_email = ""
    receiver_email = ""
 
    message = MIMEMultipart("alternative")
    message["Subject"] = "New Employees Reminder"
    message["From"] = sender_email
    message["To"] = receiver_email
 
    # Create the plain-text and HTML version of your message
    text = """
    Test
    """
    #Open the HTML file we created because we don't want to output raw HTML code. This adds the new employees table to the email.
    with open('employees.html') as fp:
        html = fp.read()
 
    part1 = MIMEText(text, "plain")
    part2 = MIMEText(html, "html")
    message.attach(part1)
    message.attach(part2)
 
    try:
        smtpObj = smtplib.SMTP('server')
        smtpObj.sendmail(sender_email, receiver_email, message.as_string())         
        print ("Successfully sent email")
    except SMTPException:
        print ("Error: unable to send email")
 
#Read the Excel file, read the specific sheet we need and take just the column we're interested in: Start Date
xls = pd.ExcelFile(r'testfile.xlsx')
df = pd.read_excel(xls, 'New Employment')
df['Start Date'] = pd.to_datetime(df['Start Date'])
today = pd.Timestamp.today()
 
#Calculate how many days are left til the employee starts working using Pandas. It's a delta from today's date and the date in the Excel file.
df['Days Until Start'] = ((df['Start Date'] - today).dt.days)
df['Days Until Start'] = df['Days Until Start'].astype(int)
delta_df = df[['Name', 'Days Until Start']]
close_delta = delta_df[delta_df['Days Until Start'].between(0,10)] #Make sure the employees delta is no bigger than 0 because that means he already started working and smaller than 10 (reasonable time to prepare for a new employee)
close_delta.to_html('employees.html') #Output the data into a table inside the HTML file which we will use for the "Send Email" function.
 
if len(close_delta) > 0: #Check if there are any employees queued up, so it won't send an email with an empty list in case there are no new employees soon.
    sendEmail()
```

The output looks something like this:

![](images/pandas.png)

Hope this will be helpful to someone 😊
