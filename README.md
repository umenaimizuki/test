# test
from slackbot.bot import listen_to
import test
from jira import JIRA
from jira.exceptions import JIRAError
import gspread
import json
import datetime

#@infra_shoyoもしくは@infra_cloudのメンションがついた際にタスクに登録
@listen_to('infra_shoyo')
@listen_to('infra_cloud')
def reaction(message):
    text = message.body['text']
    #問診票を利用した(botからの)投稿なのか、直接チャンネルに投稿されたかで処理が異なる
    if 'インフラ問診票' in text:
        sep = '`'
        t = text.split(sep)
        send_user = t[1]
    else:
        send_user = message.channel._client.users[message.body['user']][u'name']
    #ServiceAccountCredentials：Googleの各サービスへアクセスできるservice変数を生成する。
    from oauth2client.service_account import ServiceAccountCredentials
    #2つのAPIを記述しないとリフレッシュトークンを3600秒毎に発行し続けなければならない
    scope = ['https://spreadsheets.google.com/feeds','https://www.googleapis.com/auth/drive']
    #認証情報設定
    #ダウンロードしたjsonファイル名をクレデンシャル変数に設定（秘密鍵、Pythonファイルから読み込みしやすい位置に置く）
    credentials = ServiceAccountCredentials.from_json_keyfile_name('xxxx.json', scope)
    #OAuth2の資格情報を使用してGoogle APIにログインします。
    gc = gspread.authorize(credentials)
    #共有設定したスプレッドシートキーを変数[SPREADSHEET_KEY]に格納する。
    SPREADSHEET_KEY = 'xxxxxxxxxxxxxxxxxxxxxxxxxxx'
    #共有設定したスプレッドシートの一番左のシートを開く(今月の運用担当シート)
    worksheet = gc.open_by_key(SPREADSHEET_KEY).sheet1
    dt_now = datetime.datetime.now()
    day = dt_now.day + 1
    #今日の日付の担当者の値を受け取る
    import_value = worksheet.cell(day, 4).value
    #Jiraにタスクを登録
    options = {'server': 'https://adways.atlassian.net/'}
    #Jiraのアドレスとトークン
    user='xxxxxxxx@adways.net'
    token='xxxxxxxxxxxxxx'
    jira = JIRA(options, basic_auth=(user,token) )
    #Spreadsheetから取得した担当者からJiraに登録するアカウントIDを設定する。
    smr = '相談対応(' + send_user + ')_インフラに相談チャンネル'
    #pythonではswitch文を用いた分岐処理ができないため、if,elifで処理する。
    if '担当者名' in import_value:
        accountId = 'Jiraで利用しているアカウント名'
    elif '担当者名' in import_value:
        accountId = 'Jiraで利用しているアカウント名'
    elif '担当者名' in import_value:
        accountId = 'Jiraで利用しているアカウント名'
    elif '担当者名' in import_value:
        accountId = 'Jiraで利用しているアカウント名'
    else :
        accountId = ''

    new_issue = jira.create_issue(
        project='プロジェクト名',
        summary= smr,
        description= text,
        issuetype={'name': 'タスク'},
        assignee={'accountId': accountId},
        customfield_12060={'value': '相談対応'},
        customfield_12079=2
    )
