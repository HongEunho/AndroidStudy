# Google Calender API 분석

Google Calendar API는 Acl, ***CalendarList, Calendars***, Channels, ***Colors, Events***, Freebusy, Settings로 구성되어 있다.

이번 프로젝트에서 사용하게 될 API는 CalendarList, Calendars, Colors, Events 이다.

</br>

## CalendarList

- 하나의 구글계정이 갖고 있는 캘린더들의 목록을 보여주는 API이다.
- list를 통해 전체의 캘린더 목록을 가져올 수도 있고, get을 통해 하나의 목록만을 가져올 수도 있다.

### CalendarList: list

CalendarList: list는 특정 구글 계정의 캘린더 목록을 불러오는 API 이다.

구글캘린더는 한 계정에 여러개의 캘린더들을 만들 수 있는데 만들어진 캘린더의 목록들을 불러오는 것이다.

#### Request

HTTP Request는 다음과 같다.

```
GET https://www.googleapis.com/calendar/v3/users/me/calendarList
```

</br>

CalendarList: list API를 요청하기 위한 파라미터는 다음과 같다.

필수 파라미터는 없고 모두 선택적 파라미터이다.

| Parameter name  | Value     | Description                                                  |
| :-------------- | :-------- | :----------------------------------------------------------- |
| `maxResults`    | `integer` | Maximum number of entries returned on one result page. By default the value is 100 entries. The page size can never be larger than 250 entries. Optional. |
| `minAccessRole` | `string`  | The minimum access role for the user in the returned entries. Optional. The default is no restriction.   Acceptable values are:"`freeBusyReader`": The user can read free/busy information."`owner`": The user can read and modify events and access control lists."`reader`": The user can read events that are not private."`writer`": The user can read and modify events. |
| `pageToken`     | `string`  | Token specifying which result page to return. Optional.      |
| `showDeleted`   | `boolean` | Whether to include deleted calendar list entries in the result. Optional. The default is False. |
| `showHidden`    | `boolean` | Whether to show hidden entries. Optional. The default is False. |
| `syncToken`     | `string`  | 이전 list 요청 결과의 마지막 페이지에 반환된 nextSyncToken 필드에서 얻은 토큰이다. 이 파라미터를 사용하게 되면 토큰값 이후로 변경된 항목만 포함된다.</br> 즉, 특정 토큰을 파라미터로 넣게 되면 그 토큰 생성 시점 이후로 변경된 항목만 포함이 된다. showDeleted와 showHidden을 False로 설정할 수 없으며 삭제되고 숨겨진 모든 항목은 항상 결과 세트에 포함된다. 또한, 클라이언트 상태의 일관성을 보장하기 위해 minAccessRole 매개변수와 함께 사용할 수 없다. |

</br>

#### Response

| Property name   | Value    | Description                                                  | Notes |
| :-------------- | :------- | :----------------------------------------------------------- | :---- |
| `kind`          | `string` | Type of the collection ("`calendar#calendarList`").          |       |
| `etag`          | `etag`   | ETag of the collection.                                      |       |
| `nextPageToken` | `string` | Token used to access the next page of this result. Omitted if no further results are available, in which case `nextSyncToken` is provided. |       |
| `items[]`       | `list`   | Calendars that are present on the user's calendar list.      |       |
| `nextSyncToken` | `string` | Token used at a later point in time to retrieve only the entries that have changed since this result was returned. Omitted if further results are available, in which case `nextPageToken` is provided. |       |

</br>

### CalendarList: get

CalendarList: get은 캘린더 목록 중 특정 캘린더를 지정하여 불러오는 API이다.

Calendars와 헷갈릴 수 있는데, 그 차이점을 구글에서는 다음과 같이 표기하고 있다.

#### Calendars vs CalendarList

| Operation    | Calendars                                                    | CalendarList                                                 |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| insert       | Creates a new secondary calendar. By default, this calendar is also added to the creator's calendar list. | Inserts an existing calendar into the user's list            |
| delete       | Deletes a secondary calendar.                                | Removes a calendar from the user's list                      |
| get          | Retrieves calendar metadaga e.g. title, time zone.           | Retrieves metadata plus user-specific customization such as color or override reminders. |
| patch/update | Modifies calendar metadata.(description, location, summary, timezone) | Modifies user-specific calendar properties.                  |

Calendars는 메타 데이터가 주를 이룬다면, CalendarList는 메타데이터와 더불어 color와 reminder등 사용자별 특징적인 항목들 까지 갖추고 있다.



#### Request

CalendarList: get의 HTTP Request는 다음과 같다.

```
GET https://www.googleapis.com/calendar/v3/users/me/calendarList/calendarId
```

파라미터는 필수 파라미터인 ***calendarId*** 만 존재한다.

calendarId를 통해 특정 캘린더를 선택하여 그 캘린더의 정보만을 가져오게 된다.

</br>

#### Response

| Property name                                           | Value           | Description                                                  | Notes    |
| :------------------------------------------------------ | :-------------- | :----------------------------------------------------------- | :------- |
| `accessRole`                                            | `string`        | "`freeBusyReader`" - Provides read access to free/busy information.</br>"`reader`" - Provides read access to the calendar. Private events will appear to users with reader access, but event details will be hidden.</br>"`writer`" - Provides read and write access to the calendar. Private events will appear to users with writer access, and event details will be visible.</br>"`owner`" - Provides ownership of the calendar. This role has all of the permissions of the writer role with the additional ability to see and manipulate ACLs. |          |
| `backgroundColor`                                       | `string`        | The main color of the calendar in the hexadecimal format "`#0088aa`". This property supersedes the index-based `colorId`property. To set or change this property, you need to specify `colorRgbFormat=true` in the parameters of the [insert](https://developers.google.com/calendar/v3/reference/calendarList/insert), [update](https://developers.google.com/calendar/v3/reference/calendarList/update)and [patch](https://developers.google.com/calendar/v3/reference/calendarList/patch) methods. Optional. | writable |
| `colorId`                                               | `string`        | The color of the calendar. This is an ID referring to an entry in the `calendar` section of the colors definition (see the [colors endpoint](https://developers.google.com/calendar/v3/reference/colors)). This property is superseded by the `backgroundColor`and `foregroundColor` properties and can be ignored when using these properties. Optional. | writable |
| `conferenceProperties`                                  | `nested object` | Conferencing properties for this calendar, for example what types of conferences are allowed. |          |
| `conferenceProperties.allowedConferenceSolutionTypes[]` | `list`          | The types of conference solutions that are supported for this calendar. The possible values are: `"eventHangout"``"eventNamedHangout"``"hangoutsMeet"`Optional. |          |
| `defaultReminders[]`                                    | `list`          | The default reminders that the authenticated user has for this calendar. | writable |
| `defaultReminders[].method`                             | `string`        | The method used by this reminder. Possible values are: "`email`" - Reminders are sent via email."`popup`" - Reminders are sent via a UI popup.Required when adding a reminder. | writable |
| `defaultReminders[].minutes`                            | `integer`       | Number of minutes before the start of the event when the reminder should trigger. Valid values are between 0 and 40320 (4 weeks in minutes). Required when adding a reminder. | writable |
| `deleted`                                               | `boolean`       | Whether this calendar list entry has been deleted from the calendar list. Read-only. Optional. The default is False. |          |
| `description`                                           | `string`        | Description of the calendar. Optional. Read-only.            |          |
| `etag`                                                  | `etag`          | ETag of the resource.                                        |          |
| `foregroundColor`                                       | `string`        | The foreground color of the calendar in the hexadecimal format "`#ffffff`". This property supersedes the index-based `colorId`property. To set or change this property, you need to specify `colorRgbFormat=true` in the parameters of the [insert](https://developers.google.com/calendar/v3/reference/calendarList/insert), [update](https://developers.google.com/calendar/v3/reference/calendarList/update)and [patch](https://developers.google.com/calendar/v3/reference/calendarList/patch) methods. Optional. | writable |
| `hidden`                                                | `boolean`       | Whether the calendar has been hidden from the list. Optional. The attribute is only returned when the calendar is hidden, in which case the value is `true`. | writable |
| `id`                                                    | `string`        | Identifier of the calendar.                                  |          |
| `kind`                                                  | `string`        | Type of the resource ("calendar#calendarListEntry").         |          |
| `location`                                              | `string`        | Geographic location of the calendar as free-form text. Optional. Read-only. |          |
| `notificationSettings`                                  | `object`        | The notifications that the authenticated user is receiving for this calendar. | writable |
| `notificationSettings.notifications[]`                  | `list`          | The list of notifications set for this calendar.             |          |
| `notificationSettings.notifications[].method`           | `string`        | The method used to deliver the notification. The possible value is: "`email`" - Notifications are sent via email.Required when adding a notification. | writable |
| `notificationSettings.notifications[].type`             | `string`        | The type of notification. Possible values are: "`eventCreation`" - Notification sent when a new event is put on the calendar."`eventChange`" - Notification sent when an event is changed."`eventCancellation`" - Notification sent when an event is cancelled."`eventResponse`" - Notification sent when an attendee responds to the event invitation."`agenda`" - An agenda with the events of the day (sent out in the morning).Required when adding a notification. | writable |
| `primary`                                               | `boolean`       | Whether the calendar is the primary calendar of the authenticated user. Read-only. Optional. The default is False. |          |
| `selected`                                              | `boolean`       | Whether the calendar content shows up in the calendar UI. Optional. The default is False. | writable |
| `summary`                                               | `string`        | Title of the calendar. Read-only.                            |          |
| `summaryOverride`                                       | `string`        | The summary that the authenticated user has set for this calendar. Optional. | writable |
| `timeZone`                                              | `string`        | The time zone of the calendar. Optional. Read-only.          |          |

</br>

------



## Calendars

- 캘린더 하나에 대한 API로써, metadata(summary, time zone, location etc)등을 포함한다.
- 각각의 캘린더 ID는 이메일 주소이며, 하나의 캘린더는 여러 개의 Owner를 가질 수 있다.

#### Calendars: get

Calendars: get API는 특정 캘린더에 대한 메타데이터들을 가져오는 API이다.

위에서 CalendarList의 get과 비교한 것 처럼. Calendars:get에서는 메타데이터를 위주로, CalendarList:get은 메타데이터와 더불어 사용자 특정적인 정보들까지 가져오게 된다.

</br>

#### Request

HTTP Request는 다음과 같다.

```
GET https://www.googleapis.com/calendar/v3/users/me/calendarList/calendarId
```

파라미터는 CalendarList와 마찬가지로 필수 파라미터인 ***calendarId*** 하나이다.

</br>

#### Response

| Property name                                           | Value           | Description                                                  | Notes    |
| :------------------------------------------------------ | :-------------- | :----------------------------------------------------------- | :------- |
| `conferenceProperties`                                  | `nested object` | Conferencing properties for this calendar, for example what types of conferences are allowed. |          |
| `conferenceProperties.allowedConferenceSolutionTypes[]` | `list`          | The types of conference solutions that are supported for this calendar. The possible values are: `"eventHangout"``"eventNamedHangout"``"hangoutsMeet"`Optional. |          |
| `description`                                           | `string`        | Description of the calendar. Optional.                       | writable |
| `etag`                                                  | `etag`          | ETag of the resource.                                        |          |
| `id`                                                    | `string`        | Identifier of the calendar. To retrieve IDs call the [calendarList.list()](https://developers.google.com/calendar/v3/reference/calendarList/list)method. |          |
| `kind`                                                  | `string`        | Type of the resource ("`calendar#calendar`").                |          |
| `location`                                              | `string`        | Geographic location of the calendar as free-form text. Optional. | writable |
| `summary`                                               | `string`        | Title of the calendar.                                       | writable |
| `timeZone`                                              | `string`        | The time zone of the calendar. (Formatted as an IANA Time Zone Database name, e.g. "Europe/Zurich".) Optional. | writable |

## Colors

- Colors는 Calendar와 Event에 사용되는 Color들을 모아놓은 API이다.

  직접 파라미터를 주어 값을 받지는 않으며

  단순히 HTTP Request를 통해 Color 값들을 받을 수 있다.

  ```
  GET https://www.googleapis.com/calendar/v3/colors
  ```

- Calendar와 Event각각의 Background와 Foreground에 사용되는 값을 나누어 받음으로써

  캘린더UI를 꾸밀 때 참고할 수 있다.

```json
{
  "kind": "calendar#colors",
  "updated": datetime,
  "calendar": {
    (key): {
      "background": string,
      "foreground": string
    }
  },
  "event": {
    (key): {
      "background": string,
      "foreground": string
    }
  }
}
```

이러한 형식으로 Response 를 받는다.

</br>

## Events

- 이벤트(일정)과 관련된 API로써, 스케줄의 생성, 업데이트 시간과 스케줄의 시작. 종료 시간 등을 포함한다.

- 현재 프로젝트에서는 하나의 캘린더에 대한 스케줄들의 간략한 정보를 불러오는 list 메소드와

  하나의 스케줄에 대한 상세 정보를 불러오는 get 메소드를 이용할 것이다.

<br>

### Events: List

Events: list는 특정 캘린더에 대한 스케줄들의 정보를 불러오는 API 이다.

#### Request

HTTP Request는 다음과 같다.

```
GET https://www.googleapis.com/calendar/v3/calendars/calendarId/events
```

</br>

Events: list API를 요청하기 위해 필요한 파라미터는 다음과 같다.

</br>

필수 파라미터는 ***calendarId***이다. 이 calendarId가 있어야 어떤 캘린더의 이벤트들을 불러올 것인지를 정의할 수 있기 때문이다.

옵션 쿼리 파라미터는 다음과 같다. 다음 중 필요한 파라미터를 골라 호출 시 사용한다.

| Parameter name            | Value      | Description                                                  |
| ------------------------- | ---------- | ------------------------------------------------------------ |
| `iCalUID`                 | `string`   | Specifies event ID in the iCalendar format to be included in the response. Optional. |
| `maxAttendees`            | `integer`  | Response에 포함할 최대 Attendees 수이다. 지정된 Attendees보다 많을 경우에는 오직 participant만 반환한다. |
| `maxResults`              | `integer`  | 하나의 결과 페이지에 반환되는 최대 이벤트 수이다. 따라서 결과 페이지의 이벤트 수는 이 값보다 작거나 없을 수 있다.</br> 기본값은 250이며 페이지 크기는 2500보다 클 수 없다. |
| `orderBy`                 | `string`   | 결과에 반환 될 이벤트의 순서이다. 기본값은 unspecified한 안정적인 순서이며. "**startTime"**(시작 시간)을 오름차순으로 정렬하는 옵션과 **"updated"**(마지막 수정시간)을 오름차순으로 정렬하는 옵션이 있다. 단, startTime의 경우에는 singleEvent에서만 사용이 가능하다. |
| `pageToken`               | `string`   | 반환할 결과 페이지(특정 페이지)를 지정하는 토큰이다.         |
| `privateExtendedProperty` | `string`   | 오직 private properties와 매칭시키는 확장 속성 제약 조건이다. |
| `q`                       | `string`   | "q"(Free text terms)와 일치하는 이벤트를 찾기 위한 조건이다. (확장 속성은 제외한다.) |
| `sharedExtendedProperty`  | `string`   | 오직 shared properties와 매칭시키는 확장 속성 제약 조건이다. |
| `showDeleted`             | `boolean`  | 삭제된 이벤트(취소된)도 결과에 포함할지를 결정하는 조건이다. showDeleted와 singleEvents가 모두 False인 경우 삭제된 반복이벤트(기본 반복은 X)는 포함하여, 모두 True인 경우에는 삭제된 이벤트의 single Instance만 반환한다. 기본값은 False이다. |
| `showHiddenInvitations`   | `boolean`  | 숨겨진 invitations를 결과에 포함할지 결정한다. 기본값은 False. |
| `singleEvents`            | `boolean`  | 반복 이벤트를 인스턴스로 확장하고 단일 일회성 이벤트와 반복 이벤트의 인스턴스만 반환할지의 여부를 결정한다. 기본 반복이벤트는 반환하지 않는다. 기본값은 False. |
| `syncToken`               | `string`   | nextSyncToken 필드에서 얻은 토큰을 넣는 곳이다. 이전 목록 요청 결과의 마지막 페이지에 반환된 nextSyncToken 필드에서 얻은 토큰을 넣는다. 이 요청의 결과에는 그 이후로 변경된 항목만 포함된다. 이전 요청 이후 삭제된 모든 이벤트는 항상 결과에 포함되어 있으며 showDeleted를 false로 설정할 수 없다. 단, 이전과의 일관성을 유지하기 위해</br> iCalUID, orderBy, privateExtendedProperty, q, sharedExtendedProperty, timeMin, timeMax, updatedMin 속성과는 함께 사용할 수 없다. |
| `timeMax`                 | `datetime` | 필터링 할 이벤트 시작 시간의 상한점이다. timeMax 보다 큰 시작 시간을 가진 이벤트는 불러오지 않으며, 당연히 timeMin 값보다는 항상 커야한다. 형식은 RFC3339(2021-07-26) 으로 제공해야 한다. 기본값은 null로써, 시간을 지정하지 않는다. |
| `timeMin`                 | `datetime` | 필터링할 이벤트 종료 시간의 하한점이다. timeMin 보다 작은 종료 시간을 가진 이벤트는 불러오지 않으며, 당연히 timeMax 값보다는 항상 작아야 한다. 형식은 RFC3339(2021-07-26) 으로 제공해야 한다. 기본값은 null로써, 시간을 지정하지 않는다. |
| `timeZone`                | `string`   | Response에 사용될 달력의 시간대이다. 기본값은 원래 달력의 시간대이다. |
| `updatedMin`              | `datetime` | 필터링할 이벤트 마지막 수정시간의 하한점이다. 이 속성을 사용하면 showDeleted와는 상관없이 timeMin 보다 큰 update 시간을 가진 이벤트는 불러오게 된다. 형식은 RFC3339(2021-07-26) 으로 제공해야 한다. 기본값은 null로써, 시간을 지정하지 않는다. |

</br>

#### Response

Response 속성과 각각의 역할은 다음과 같다.

| Property name                | Value      | Description                                                  | Notes    |
| :--------------------------- | :--------- | :----------------------------------------------------------- | :------- |
| `kind`                       | `string`   | Type of the collection ("`calendar#events`").                |          |
| `etag`                       | `etag`     | ETag of the collection.                                      |          |
| `summary`                    | `string`   | Title of the calendar. Read-only.                            |          |
| `description`                | `string`   | Description of the calendar. Read-only.                      |          |
| `updated`                    | `datetime` | Last modification time of the calendar (as a [RFC3339](https://tools.ietf.org/html/rfc3339) timestamp). Read-only. |          |
| `timeZone`                   | `string`   | The time zone of the calendar. Read-only.                    |          |
| `accessRole`                 | `string`   | The user's access role for this calendar. Read-only. Possible values are: "`none`" - The user has no access."`freeBusyReader`" - The user has read access to free/busy information."`reader`" - The user has read access to the calendar. Private events will appear to users with reader access, but event details will be hidden."`writer`" - The user has read and write access to the calendar. Private events will appear to users with writer access, and event details will be visible."`owner`" - The user has ownership of the calendar. This role has all of the permissions of the writer role with the additional ability to see and manipulate ACLs. |          |
| `defaultReminders[]`         | `list`     | The default reminders on the calendar for the authenticated user. These reminders apply to all events on this calendar that do not explicitly override them (i.e. do not have `reminders.useDefault` set to True). |          |
| `defaultReminders[].method`  | `string`   | The method used by this reminder. Possible values are: "`email`" - Reminders are sent via email."`popup`" - Reminders are sent via a UI popup. Required when adding a reminder. | writable |
| `defaultReminders[].minutes` | `integer`  | Number of minutes before the start of the event when the reminder should trigger. Valid values are between 0 and 40320 (4 weeks in minutes). Required when adding a reminder. | writable |
| `nextPageToken`              | `string`   | Token used to access the next page of this result. Omitted if no further results are available, in which case `nextSyncToken` is provided. |          |
| `items[]`                    | `list`     | List of events on the calendar. (이벤트들의 항목들이 표시된다.) |          |
| `nextSyncToken`              | `string`   | Token used at a later point in time to retrieve only the entries that have changed since this result was returned. Omitted if further results are available, in which case `nextPageToken` is provided. |          |

</br>

### Events: get

Events: get는 특정 이벤트(스케줄)에 대한 상세 정보를 불러오는 API 이다.

#### Request

HTTP Request는 다음과 같다.

```
GET https://www.googleapis.com/calendar/v3/calendars/calendarId/events/eventId
```

</br>

Events: get API를 요청하기 위해 필요한 파라미터는 다음과 같다.

**calendarId**와 **eventId**는 필수 파라미터이다.

어떤 캘린더안의 어떤 이벤트인지에 대해 필수로 알아야 그 이벤트에 대한 정보를 불러올 수 있기 때문이다.

옵션 파라미터는 다음과 같다.

| Parameter name       | Value     | Description                                                  |
| :------------------- | :-------- | :----------------------------------------------------------- |
| `alwaysIncludeEmail` | `boolean` | Deprecated and ignored.</br> 실제 이메일 주소를 사용할 수 없는 경우에도 organizer, creator, attendees의 이메일 필드에 항상 값이 반환된다. |
| `maxAttendees`       | `integer` | Response에 포함할 최대 참석자 수이다.                        |
| `timeZone`           | `string`  | Time zone used in the response. Optional. The default is the time zone of the calendar. |

</br>

#### Response

Response 속성과 각각의 역할은 다음과 같다.

| Property name                  | Value     | Description                                                  | Notes    |
| ------------------------------ | --------- | ------------------------------------------------------------ | -------- |
| `anyoneCanAddSelf`             | `boolean` | Deprecated. </br> 누구든지 이벤트에 초대할 수 있는지 여부. 기본값은 False | Writable |
| ***`attachments[]`***          | `list`    | 이벤트 첨부파일로써, 현재는 구글 드라이브 첨부 파일만 지원된다. 이벤트당 최대 25개의 첨부파일이 가능하며 첨부 파일을 수정하려면 supportAttachments 매개변수를 true로 설정해야 한다. |          |
| `attachments[].fileId`         | `string`  | 첨부 파일의 ID                                               |          |
| `attachments[].fileUrl`        | `string`  | URL link to the attachment. For adding Google Drive file attachments use the same format as in `alternateLink` property of the `Files` resource in the Drive API. Required when adding an attachment. | Writable |
| `attachments[].iconLink`       | `string`  | URL link to the attachment's icon. Read-only.                |          |
| `attachments[].mimeType`       | `string`  | Internet media type (MIME type) of the attachment.           |          |
| `attachments[].title`          | `string`  | Attachment title.                                            |          |
| `attendeesOmitted`             | `boolean` | Attendees가 이벤트에서 누락되었는지에 대한 여부. When retrieving an event, this may be due to a restriction specified by the `maxAttendee` query parameter. When updating an event, this can be used to only update the participant's response. Optional. The default is False. | Writable |
| ***`attendees[]`***            | *`list`*  | The attendees of the event. See the [Events with attendees](https://developers.google.com/calendar/concepts/sharing) guide for more information on scheduling events with other calendar users. Service accounts need to use [domain-wide delegation of authority](https://developers.google.com/calendar/auth#perform-g-suite-domain-wide-delegation-of-authority) to populate the attendee list. | Writable |
| `attendees[].additionalGuests` | `integer` | Number of additional guests. Optional. The default is 0.     | Writable |
| `attendees[].comment`          | `string`  | The attendee's response comment. Optional.                   | Writable |
| `attendees[].displayName`      | `string`  | The attendee's name, if available. Optional.                 | Writable |
| `attendees[].email`            | `string`  | The attendee's email address, if available. This field must be present when adding an attendee. It must be a valid email address as per [RFC5322](https://tools.ietf.org/html/rfc5322#section-3.4). Required when adding an attendee. |          |
| `attendees[].id`               | `string`  | The attendee's Profile ID, if available.                     |          |

| **`attendees[].optional`**                                | **`boolean`**     | 선택적 참석자인지에 대한 여부. 기본값은 False                | writable |
| --------------------------------------------------------- | ----------------- | ------------------------------------------------------------ | -------- |
| `attendees[].organizer`                                   | `boolean`         | attendees가 이벤트의 주최자인지. 기본값은 False              |          |
| `attendees[].resource`                                    | `boolean`         | Whether the attendee is a resource. 참석자가 이벤트에 처음 추가된 경우에만 사용 가능. Subsequent modifications are ignored. The default is False. | writable |
| `attendees[].responseStatus`                              | `string`          | 참석자의 초대에 대한 응답상태. </br>Possible values are: "`needsAction`" - 아직 응답하지 않음</br>""`declined`" - 초대 거절 </br>"`tentative`" - 임시 수락"</br>`accepted`" - 수락 | writable |
| `attendees[].self`                                        | `boolean`         | Whether this entry represents the calendar on which this copy of the event appears. Read-only. The default is False. |          |
| `colorId`                                                 | `string`          | The color of the event. This is an ID referring to an entry in the `event` section of the colors definition (see the [colors endpoint](https://developers.google.com/calendar/v3/reference/colors)). Optional. | writable |
| ***`conferenceData`***                                    | *`nested object`* | Google Meet 회의 세부정보와 같은 회의 관련 정보를 나타냄. 새 회의정보를 만드려면 createRequest 필드를 사용해야 함. 변경사항을 유지하려면 모든 이벤트 수정 요청에 대해 conferenceDataVersion 요청 매개변수에 1로 설정해야함. | writable |
| `conferenceData.conferenceId`                             | `string`          | The ID of the conference. Can be used by developers to keep track of conferences, should not be displayed to users. The ID value is formed differently for each conference solution type: `eventHangout`: ID is not set. `eventNamedHangout`: ID is the name of the Hangout. `hangoutsMeet`: ID is the 10-letter meeting code, for example `aaa-bbbb-ccc`. `addOn`: ID is defined by the third-party provider. Optional. |          |
| `conferenceData.conferenceSolution`                       | `nested object`   | 행아웃이나 구글미트 같은 회의 솔루션. 하나 이상의 createRequest 또는 entryPoint가 필수로 필요함. |          |
| `conferenceData.conferenceSolution.iconUri`               | `string`          | The user-visible icon for this solution.                     |          |
| `conferenceData.conferenceSolution.key`                   | `nested object`   | The key which can uniquely identify the conference solution for this event. |          |
| `conferenceData.conferenceSolution.key.type`              | `string`          | The conference solution type. If a client encounters an unfamiliar or empty type, it should still be able to display the entry points. However, it should disallow modifications. The possible values are: `"eventHangout"` for Hangouts for consumers (http://hangouts.google.com) `"eventNamedHangout"` for classic Hangouts for Google Workspace users (deprecated; http://hangouts.google.com) `"hangoutsMeet"` for Google Meet (http://meet.google.com) `"addOn"` for 3P conference providers |          |
| `conferenceData.conferenceSolution.name`                  | `string`          | 사용자에게 표시되는 회의솔루션의 이름                        |          |
| `conferenceData.createRequest`                            | `nested object`   | 새 회의를 생성하고 이벤트에 첨부하라는 요청. 하나 이상의 entryPoint 또는 createRequest가 필요함. |          |
| `conferenceData.createRequest.conferenceSolutionKey`      | `nested object`   | The conference solution, such as Hangouts or Google Meet.    |          |
| `conferenceData.createRequest.conferenceSolutionKey.type` | `string`          | The conference solution type. If a client encounters an unfamiliar or empty type, it should still be able to display the entry points. However, it should disallow modifications. The possible values are: `"eventHangout"` for Hangouts for consumers (http://hangouts.google.com) `"eventNamedHangout"` for classic Hangouts for Google Workspace users (deprecated; http://hangouts.google.com) `"hangoutsMeet"` for Google Meet (http://meet.google.com) `"addOn"` for 3P conference providers |          |
| `conferenceData.createRequest.requestId`                  | `string`          | 클라이언트가 생성한 고유ID</br> 클라이언트는 모든 새로운 요청에 대해 이 ID를 다시 생성해야 함. |          |
| `conferenceData.createRequest.status`                     | `nested object`   | 회의 생성요청의 상태                                         |          |
| `conferenceData.createRequest.status.statusCode`          | `string`          | The current status of the conference create request. Read-only. The possible values are: `"pending"`: the conference create request is still being processed. `"success"`: the conference create request succeeded, the entry points are populated. `"failure"`: the conference create request failed, there are no entry points. |          |
| `conferenceData.entryPoints[]`                            | `list`            | URL이나 전화번호 같은 회의 진입점에 필요한 정보. ConferenceSolution과 하나 이상의 entryPoint or createRequest가 필요 |          |
| `conferenceData.entryPoints[].accessCode`                 | `string`          | The access code to access the conference. The maximum length is 128 characters. When creating new conference data, populate only the subset of {`meetingCode`, `accessCode`, `passcode`, `password`, `pin`} fields that match the terminology that the conference provider uses. Only the populated fields should be displayed. Optional. |          |
| `conferenceData.entryPoints[].entryPointType`             | `string`          | The type of the conference entry point. Possible values are: `"video"` - joining a conference over HTTP. A conference can have zero or one `video` entry point. `"phone"` - joining a conference by dialing a phone number. A conference can have zero or more `phone` entry points. `"sip"` - joining a conference over SIP. A conference can have zero or one `sip` entry point. `"more"` - further conference joining instructions, for example additional phone numbers. A conference can have zero or one `more` entry point. A conference with only a `more` entry point is not a valid conference. |          |
| `conferenceData.entryPoints[].label`                      | `string`          | The label for the URI. Visible to end users. Not localized. The maximum length is 512 characters. Examples: for `video`: meet.google.com/aaa-bbbb-ccc for `phone`: +1 123 268 2601 for `sip`: 12345678@altostrat.com for `more`: should not be filled Optional. |          |
| `conferenceData.entryPoints[].meetingCode`                | `string`          | The meeting code to access the conference. The maximum length is 128 characters. When creating new conference data, populate only the subset of {`meetingCode`, `accessCode`, `passcode`, `password`, `pin`} fields that match the terminology that the conference provider uses. Only the populated fields should be displayed. Optional. |          |
| `conferenceData.entryPoints[].passcode`                   | `string`          | The passcode to access the conference. The maximum length is 128 characters. When creating new conference data, populate only the subset of {`meetingCode`, `accessCode`, `passcode`, `password`, `pin`} fields that match the terminology that the conference provider uses. Only the populated fields should be displayed. |          |
| `conferenceData.entryPoints[].password`                   | `string`          | The password to access the conference. The maximum length is 128 characters. When creating new conference data, populate only the subset of {`meetingCode`, `accessCode`, `passcode`, `password`, `pin`} fields that match the terminology that the conference provider uses. Only the populated fields should be displayed. Optional. |          |
| `conferenceData.entryPoints[].pin`                        | `string`          | The PIN to access the conference. The maximum length is 128 characters. When creating new conference data, populate only the subset of {`meetingCode`, `accessCode`, `passcode`, `password`, `pin`} fields that match the terminology that the conference provider uses. Only the populated fields should be displayed. Optional. |          |
| `conferenceData.entryPoints[].uri`                        | `string`          | The URI of the entry point. The maximum length is 1300 characters. Format: for `video`, `http:` or `https:` schema is required. for `phone`, `tel:` schema is required. The URI should include the entire dial sequence (e.g., tel:+12345678900,,,123456789;1234). for `sip`, `sip:` schema is required, e.g., sip:12345678@myprovider.com. for `more`, `http:` or `https:` schema is required. |          |
| `conferenceData.notes`                                    | `string`          | Additional notes (such as instructions from the domain administrator, legal notices) to display to the user. Can contain HTML. The maximum length is 2048 characters. Optional. |          |
| `conferenceData.signature`                                | `string`          | The signature of the conference data. Generated on server side. Must be preserved while copying the conference data between events, otherwise the conference data will not be copied. Unset for a conference with a failed create request. Optional for a conference with a pending create request. |          |
| `created`                                                 | `datetime`        | Creation time of the event (as a [RFC3339](https://tools.ietf.org/html/rfc3339) timestamp). Read-only. |          |
| ***`creator`***                                           | *`object`*        | The creator of the event. Read-only.                         |          |
| `creator.displayName`                                     | `string`          | The creator's name, if available.                            |          |
| `creator.email`                                           | `string`          | The creator's email address, if available.                   |          |
| `creator.id`                                              | `string`          | The creator's Profile ID, if available.                      |          |
| `creator.self`                                            | `boolean`         | Whether the creator corresponds to the calendar on which this copy of the event appears. Read-only. The default is False. |          |
| `description`                                             | `string`          | Description of the event. Can contain HTML. Optional.        | writable |
| ***`end`***                                               | *`nested object`* | The (exclusive) end time of the event. For a recurring event, this is the end time of the first instance. |          |
| `end.date`                                                | `date`            | The date, in the format "yyyy-mm-dd", if this is an all-day event. | writable |
| `end.dateTime`                                            | `datetime`        | The time, as a combined date-time value (formatted according to [RFC3339](https://tools.ietf.org/html/rfc3339)). A time zone offset is required unless a time zone is explicitly specified in `timeZone`. | writable |
| `end.timeZone`                                            | `string`          | The time zone in which the time is specified. (Formatted as an IANA Time Zone Database name, e.g. "Europe/Zurich".) For recurring events this field is required and specifies the time zone in which the recurrence is expanded. For single events this field is optional and indicates a custom time zone for the event start/end. | writable |
| `endTimeUnspecified`                                      | `boolean`         | Whether the end time is actually unspecified. An end time is still provided for compatibility reasons, even if this attribute is set to True. The default is False. |          |
| `etag`                                                    | `etag`            | ETag of the resource.                                        |          |
| `eventType`                                               | `string`          | Specific type of the event. Read-only. Possible values are: "`default`" - A regular event or not further specified."`outOfOffice`" - An out-of-office event. |          |
| `extendedProperties`                                      | `object`          | Extended properties of the event.                            |          |
| ***`extendedProperties.private`***                        | *`object`*        | Properties that are private to the copy of the event that appears on this calendar. | writable |
| `extendedProperties.private.(key)`                        | `string`          | The name of the private property and the corresponding value. |          |
| `extendedProperties.shared`                               | `object`          | Properties that are shared between copies of the event on other attendees' calendars. | writable |
| `extendedProperties.shared.(key)`                         | `string`          | The name of the shared property and the corresponding value. |          |
| ***`gadget`***                                            | *`object`*        | A gadget that extends this event. Gadgets are deprecated; this structure is instead only used for returning birthday calendar metadata. |          |
| `gadget.display`                                          | `string`          | The gadget's display mode. Deprecated. Possible values are: "`icon`" - The gadget displays next to the event's title in the calendar view."`chip`" - The gadget displays when the event is clicked. | writable |
| `gadget.height`                                           | `integer`         | The gadget's height in pixels. The height must be an integer greater than 0. Optional. Deprecated. | writable |
| `gadget.iconLink`                                         | `string`          | The gadget's icon URL. The URL scheme must be HTTPS. Deprecated. | writable |
| `gadget.link`                                             | `string`          | The gadget's URL. The URL scheme must be HTTPS. Deprecated.  | writable |
| `gadget.preferences`                                      | `object`          | Preferences.                                                 | writable |
| `gadget.preferences.(key)`                                | `string`          | The preference name and corresponding value.                 |          |
| `gadget.title`                                            | `string`          | The gadget's title. Deprecated.                              | writable |
| `gadget.type`                                             | `string`          | The gadget's type. Deprecated.                               | writable |
| `gadget.width`                                            | `integer`         | The gadget's width in pixels. The width must be an integer greater than 0. Optional. Deprecated. | writable |
| ***`guestsCanInviteOthers`***                             | *`boolean`*       | Whether attendees other than the organizer can invite others to the event. Optional. The default is True. | writable |
| `guestsCanModify`                                         | `boolean`         | Whether attendees other than the organizer can modify the event. Optional. The default is False. | writable |
| `guestsCanSeeOtherGuests`                                 | `boolean`         | Whether attendees other than the organizer can see who the event's attendees are. Optional. The default is True. | writable |
| `hangoutLink`                                             | `string`          | An absolute link to the Google Hangout associated with this event. Read-only. |          |
| `htmlLink`                                                | `string`          | An absolute link to this event in the Google Calendar Web UI. Read-only. |          |
| `iCalUID`                                                 | `string`          | Event unique identifier as defined in [RFC5545](https://tools.ietf.org/html/rfc5545#section-3.8.4.7). It is used to uniquely identify events accross calendaring systems and must be supplied when importing events via the [import](https://developers.google.com/calendar/v3/reference/events/import) method. Note that the `icalUID` and the `id` are not identical and only one of them should be supplied at event creation time. One difference in their semantics is that in recurring events, all occurrences of one event have different `id`s while they all share the same `icalUID`s. |          |
| `id`                                                      | `string`          | Opaque identifier of the event. When creating new single or recurring events, you can specify their IDs. Provided IDs must follow these rules: characters allowed in the ID are those used in base32hex encoding, i.e. lowercase letters a-v and digits 0-9, see section 3.1.2 in [RFC2938](http://tools.ietf.org/html/rfc2938#section-3.1.2)the length of the ID must be between 5 and 1024 charactersthe ID must be unique per calendarDue to the globally distributed nature of the system, we cannot guarantee that ID collisions will be detected at event creation time. To minimize the risk of collisions we recommend using an established UUID algorithm such as one described in [RFC4122](https://tools.ietf.org/html/rfc4122). If you do not specify an ID, it will be automatically generated by the server. Note that the `icalUID` and the `id` are not identical and only one of them should be supplied at event creation time. One difference in their semantics is that in recurring events, all occurrences of one event have different `id`s while they all share the same `icalUID`s. | writable |
| `kind`                                                    | `string`          | Type of the resource ("`calendar#event`").                   |          |
| `location`                                                | `string`          | Geographic location of the event as free-form text. Optional. | writable |
| `locked`                                                  | `boolean`         | Whether this is a locked event copy where no changes can be made to the main event fields "summary", "description", "location", "start", "end" or "recurrence". The default is False. Read-Only. |          |
| `organizer`                                               | `object`          | The organizer of the event. If the organizer is also an attendee, this is indicated with a separate entry in `attendees` with the `organizer` field set to True. To change the organizer, use the [move](https://developers.google.com/calendar/v3/reference/events/move) operation. Read-only, except when importing an event. | writable |
| `organizer.displayName`                                   | `string`          | The organizer's name, if available.                          | writable |
| `organizer.email`                                         | `string`          | The organizer's email address, if available. It must be a valid email address as per [RFC5322](https://tools.ietf.org/html/rfc5322#section-3.4). | writable |
| `organizer.id`                                            | `string`          | The organizer's Profile ID, if available.                    |          |
| `organizer.self`                                          | `boolean`         | Whether the organizer corresponds to the calendar on which this copy of the event appears. Read-only. The default is False. |          |
| ***`originalStartTime`***                                 | *`nested object`* | For an instance of a ***recurring event***, this is the time at which this event would start according to the recurrence data in the recurring event identified by recurringEventId. It uniquely identifies the instance within the recurring event series even if the instance was moved to a different time. Immutable. |          |
| `originalStartTime.date`                                  | `date`            | The date, in the format "yyyy-mm-dd", if this is an all-day event. | writable |
| `originalStartTime.dateTime`                              | `datetime`        | The time, as a combined date-time value (formatted according to [RFC3339](https://tools.ietf.org/html/rfc3339)). A time zone offset is required unless a time zone is explicitly specified in `timeZone`. | writable |
| `originalStartTime.timeZone`                              | `string`          | The time zone in which the time is specified. (Formatted as an IANA Time Zone Database name, e.g. "Europe/Zurich".) For recurring events this field is required and specifies the time zone in which the recurrence is expanded. For single events this field is optional and indicates a custom time zone for the event start/end. | writable |
| `privateCopy`                                             | `boolean`         | If set to True, [Event propagation](https://developers.google.com/calendar/concepts/sharing#event_propagation) is disabled. Note that it is not the same thing as [Private event properties](https://developers.google.com/calendar/concepts/sharing#private_event_properties). Optional. Immutable. The default is False. |          |
| ***`recurrence[]`***                                      | *`list`*          | List of RRULE, EXRULE, RDATE and EXDATE lines for a recurring event, as specified in [RFC5545](http://tools.ietf.org/html/rfc5545#section-3.8.5). Note that DTSTART and DTEND lines are not allowed in this field; event start and end times are specified in the `start` and `end` fields. This field is omitted for single events or instances of recurring events. | writable |
| ***`recurringEventId`***                                  | *`string`*        | For an instance of a recurring event, this is the `id` of the recurring event to which this instance belongs. Immutable. |          |
| ***`reminders`***                                         | *`object`*        | Information about the event's reminders for the authenticated user. |          |
| `reminders.overrides[]`                                   | `list`            | If the event doesn't use the default reminders, this lists the reminders specific to the event, or, if not set, indicates that no reminders are set for this event. The maximum number of override reminders is 5. | writable |
| `reminders.overrides[].method`                            | `string`          | The method used by this reminder. Possible values are: "`email`" - Reminders are sent via email."`popup`" - Reminders are sent via a UI popup.Required when adding a reminder. | writable |
| `reminders.overrides[].minutes`                           | `integer`         | Number of minutes before the start of the event when the reminder should trigger. Valid values are between 0 and 40320 (4 weeks in minutes). Required when adding a reminder. | writable |
| `reminders.useDefault`                                    | `boolean`         | Whether the default reminders of the calendar apply to the event. | writable |
| `sequence`                                                | `integer`         | Sequence number as per iCalendar.                            | writable |
| **`source`**                                              | `object`          | Source from which the event was created. For example, a web page, an email message or any document identifiable by an URL with HTTP or HTTPS scheme. Can only be seen or modified by the creator of the event. |          |
| `source.title`                                            | `string`          | Title of the source; for example a title of a web page or an email subject. | writable |
| `source.url`                                              | `string`          | URL of the source pointing to a resource. The URL scheme must be HTTP or HTTPS. | writable |
| `start`                                                   | `nested object`   | The (inclusive) start time of the event. For a recurring event, this is the start time of the first instance. |          |
| `start.date`                                              | `date`            | The date, in the format "yyyy-mm-dd", if this is an all-day event. | writable |
| `start.dateTime`                                          | `datetime`        | The time, as a combined date-time value (formatted according to [RFC3339](https://tools.ietf.org/html/rfc3339)). A time zone offset is required unless a time zone is explicitly specified in `timeZone`. | writable |
| `start.timeZone`                                          | `string`          | The time zone in which the time is specified. (Formatted as an IANA Time Zone Database name, e.g. "Europe/Zurich".) For recurring events this field is required and specifies the time zone in which the recurrence is expanded. For single events this field is optional and indicates a custom time zone for the event start/end. | writable |
| `status`                                                  | `string`          | Status of the event. Optional. Possible values are: "`confirmed`" - The event is confirmed. This is the default status."`tentative`" - The event is tentatively confirmed."`cancelled`" - The event is cancelled (deleted). The [list](https://developers.google.com/calendar/v3/reference/events/list) method returns cancelled events only on incremental sync (when `syncToken` or `updatedMin` are specified) or if the `showDeleted` flag is set to `true`. The [get](https://developers.google.com/calendar/v3/reference/events/get) method always returns them. A cancelled status represents two different states depending on the event type: Cancelled exceptions of an uncancelled recurring event indicate that this instance should no longer be presented to the user. Clients should store these events for the lifetime of the parent recurring event. Cancelled exceptions are only guaranteed to have values for the `id`, `recurringEventId` and `originalStartTime` fields populated. The other fields might be empty. All other cancelled events represent deleted events. Clients should remove their locally synced copies. Such cancelled events will eventually disappear, so do not rely on them being available indefinitely. Deleted events are only guaranteed to have the `id` field populated. On the organizer's calendar, cancelled events continue to expose event details (summary, location, etc.) so that they can be restored (undeleted). Similarly, the events to which the user was invited and that they manually removed continue to provide details. However, incremental sync requests with `showDeleted` set to false will not return these details. If an event changes its organizer (for example via the [move](https://developers.google.com/calendar/v3/reference/events/move) operation) and the original organizer is not on the attendee list, it will leave behind a cancelled event where only the `id` field is guaranteed to be populated. | writable |
| `summary`                                                 | `string`          | Title of the event.                                          | writable |
| `transparency`                                            | `string`          | Whether the event blocks time on the calendar. Optional. Possible values are: "`opaque`" - Default value. The event does block time on the calendar. This is equivalent to setting **Show me as** to **Busy** in the Calendar UI."`transparent`" - The event does not block time on the calendar. This is equivalent to setting **Show me as** to **Available** in the Calendar UI. | writable |
| `updated`                                                 | `datetime`        | Last modification time of the event (as a [RFC3339](https://tools.ietf.org/html/rfc3339) timestamp). Read-only. |          |
| `visibility`                                              | `string`          | Visibility of the event. Optional. Possible values are: "`default`" - Uses the default visibility for events on the calendar. This is the default value."`public`" - The event is public and event details are visible to all readers of the calendar."`private`" - The event is private and only event attendees may view event details."`confidential`" - The event is private. This value is provided for compatibility reasons. | writable |

