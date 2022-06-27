
登入套件及日曆套件

```
npm i --save angularx-social-login 
npm i --save @fullcalendar/angular @fullcalendar/daygrid 
```


# 登入以使用自己的行事曆

啟用 Google Calendar API


## 建立 OAuth2.0 用來登入


![Image 1](https://user-images.githubusercontent.com/28519748/173192916-64d8c589-7236-4335-afd9-5955194183a7.png)


![Image 3](https://user-images.githubusercontent.com/28519748/173192921-917bc226-d70e-4f99-bd70-bf72c382d758.png)

![未命名](https://user-images.githubusercontent.com/28519748/173193004-eaa46014-2c27-4769-a934-17ffff2e575e.png)


## 建立 API KEY 

![未命名](https://user-images.githubusercontent.com/28519748/173193065-cb23befe-21cf-43c9-86f0-85a984a89f34.png)





# 與 google 串接

## 取得 CalendarList 

```
this.httpClient.get<any>(`https://content.googleapis.com/calendar/v3/users/me/calendarList`,
      {
        params: {
          'key': environment.googleCalendarApiKey,
          'singleEvents': true,
          'maxResults': 9999
        },
        headers: {
          'Authorization': `Bearer ${auth.authToken}`   // google登入後取得的 user token
        }
      }
    );
```



## 取得 events 


要記得 _id_, _timeMin_, _timeMax_  必須要 url encode

```

this.httpClient.get<any>(
      `https://www.googleapis.com/calendar/v3/calendars/${id}/events?timeMin=${timeMin}&timeMax=${timeMax}`,
      {
        params: {
          'key': environment.googleCalendarApiKey,
          'singleEvents': true,
          'maxResults': 9999
        },
        headers: {
          'Authorization': `Bearer ${auth.authToken}`
        }
      }
    );


```


## 取得 google 日曆的顏色設定

```

return this.httpClient.get<{
  calendar: { [key: string]: { background: string, foreground: string } },
  event: { [key: string]: { background: string, foreground: string } }
}>(
  `https://content.googleapis.com/calendar/v3/colors`,
  {
    params: {
      'key': environment.googleCalendarApiKey,
    }
  });
}

```



## fullcalendar 的 eventSources 設定

如何從 google 的 events 轉為 fullcalendar 裡的 event


```

{
  eventSources: _.map(calendarListId, cid => {
    return {
      events: (info: any, successCallback: any, failureCallback: any) => {
        this.today.eventsList(cid.id, info.startStr, info.endStr).pipe(
          map(a => a.items as any[]),
          map(a => a.map(entry => {
            return {
              id: entry.id,
              title: (entry?.summary) ?? '',
              start: entry.start.dateTime || entry.start.date, // try timed. will fall back to all-day
              end: entry.end.dateTime || entry.end.date, // same
              url: entry.htmlLink,
              location: (entry?.location) ?? '',
              description: (entry?.description) ?? '',
              // color: colorId.event[entry.colorId]?.background ?? colorId.calendar[cid.colorId].background, //  可對應 google color 取到的 color id
              // textColor: colorId.event[entry.colorId]?.foreground ?? colorId.calendar[cid.colorId].foreground,   // 可對應 google color 取到的 color id
            }
          }))).subscribe(a => successCallback(a))
      }
    }
  })
}

```


