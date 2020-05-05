# How to design and configure your appointment editor in Flutter event calendar (SfCalendar)?

In flutter event calendar, we can develop appointment editor application with custom appointment using `OnTap` callback of the calendar widget.

## Step 1:
Create a custom class Meeting with required fields (mandatory fields are from, and to).

```xml
class Meeting {
  Meeting(
      {@required this.from,
      @required this.to,
      this.background = Colors.green,
      this.isAllDay = false,
      this.eventName = '',
      this.startTimeZone = '',
      this.endTimeZone = '',
      this.description = ''});
 
  final String eventName;
  final DateTime from;
  final DateTime to;
  final Color background;
  final bool isAllDay;
  final String startTimeZone;
  final String endTimeZone;
  final String description;
}
```
## Step 2:
You can map those properties of Meeting class with calendar widget by using the ‘CalendarDataSource’ override methods properties.

```xml
Class DataSource extends CalendarDataSource {
  DataSource(List<Meeting> source) {
    appointments = source;
  }
 
  @override
  bool isAllDay(int index) => appointments[index].isAllDay;
 
  @override
  String getSubject(int index) => appointments[index].eventName;
 
  @override
  String getStartTimeZone(int index) => appointments[index].startTimeZone;
 
  @override
  String getNotes(int index) => appointments[index].description;
 
  @override
  String getEndTimeZone(int index) => appointments[index].endTimeZone;
 
  @override
  Color getColor(int index) => appointments[index].background;
 
  @override
  DateTime getStartTime(int index) => appointments[index].from;
 
  @override
  DateTime getEndTime(int index) => appointments[index].to;
}
```
 

## Step 3:
You can schedule meetings for a day by setting `From` and `To` properties of Meeting class. Create collection of `Meetings`  data and assign it to `appointments` property of `CalendarDataSource`.

```xml
List<Meeting> getMeetingDetails() {
  final List<Meeting> meetingCollection = <Meeting>[];
 
  final DateTime today = DateTime.now();
  final Random random = Random();
  for (int month = -1; month < 2; month++) {
    for (int day = -5; day < 5; day++) {
      for (int hour = 9; hour < 18; hour += 5) {
        meetingCollection.add(Meeting(
          from: today
              .add(Duration(days: (month * 30) + day))
              .add(Duration(hours: hour)),
          to: today
              .add(Duration(days: (month * 30) + day))
              .add(Duration(hours: hour + 2)),
          background: _colorCollection[random.nextInt(9)],
          startTimeZone: ‘’,
          endTimeZone: ‘’,
          description: ‘’,
          isAllDay: false,
          eventName: eventNameCollection[random.nextInt(7)],
        ));
      }
    }
  }
 
  return meetingCollection;
}
```

## Step 4:
In the appointment editor class, you can create the custom editor for appointment with required data fields.

```xml
Widget _getAppointmentEditor(BuildContext context) {
  return Container(
      color: Colors.white,
      child: ListView(
        padding: const EdgeInsets.all(0),
        children: <Widget>[
          ListTile(
            contentPadding: const EdgeInsets.fromLTRB(5, 0, 5, 5),
            leading: const Text(‘’),
            title: TextField(
              controller: TextEditingController(text: _subject),
              onChanged: (String value) {
                _subject = value;
              },
              keyboardType: TextInputType.multiline,
              maxLines: null,
              style: TextStyle(
                  fontSize: 25,
                  color: Colors.black,
                  fontWeight: FontWeight.w400),
              decoration: InputDecoration(
                border: InputBorder.none,
                hintText: ‘Add title’,
              ),
            ),
          ),
          const Divider(
            height: 1.0,
            thickness: 1,
          ),
          ListTile(
              contentPadding: const EdgeInsets.fromLTRB(5, 2, 5, 2),
              leading: Icon(
                Icons.access_time,
                color: Colors.black54,
              ),
              title: Row(children: <Widget>[
                const Expanded(
                  child: Text(‘All-day’),
                ),
                Expanded(
                    child: Align(
                        alignment: Alignment.centerRight,
                        child: Switch(
                          value: _isAllDay,
                          onChanged: (bool value) {
                            setState(() {
                              _isAllDay = value;
                            });
                          },
                        ))),
              ])),
          ListTile(
              contentPadding: const EdgeInsets.fromLTRB(5, 2, 5, 2),
              leading: const Text(‘’),
              title: Row(
                  crossAxisAlignment: CrossAxisAlignment.center,
                  children: <Widget>[
                    Expanded(
                      flex: 7,
                      child: GestureDetector(
                          child: Text(
                              DateFormat(‘EEE, MMM dd yyyy’)
                                  .format(_startDate),
                              textAlign: TextAlign.left),
                          onTap: () async {
                            final DateTime date = await showDatePicker(
                              context: context,
                              initialDate: _startDate,
                              firstDate: DateTime(1900),
                              lastDate: DateTime(2100),
                            );
 
                            if (date != null && date != _startDate) {
                              setState(() {
                                final Duration difference =
                                    _endDate.difference(_startDate);
                                _startDate = DateTime(
                                    date.year,
                                    date.month,
                                    date.day,
                                    _startTime.hour,
                                    _startTime.minute,
                                    0);
                                _endDate = _startDate.add(difference);
                                _endTime = TimeOfDay(
                                    hour: _endDate.hour,
                                    minute: _endDate.minute);
                              });
                            }
                          }),
                    ),
                    Expanded(
                        flex: 3,
                        child: _isAllDay
                            ? const Text(‘’)
                            : GestureDetector(
                                child: Text(
                                  DateFormat(‘hh:mm a’).format(_startDate),
                                  textAlign: TextAlign.right,
                                ),
                                onTap: () async {
                                  final TimeOfDay time = await showTimePicker(
                                      context: context,
                                      initialTime: TimeOfDay(
                                          hour: _startTime.hour,
                                          minute: _startTime.minute));
 
                                  if (time != null && time != _startTime) {
                                    setState(() {
                                      _startTime = time;
                                      final Duration difference =
                                          _endDate.difference(_startDate);
                                      _startDate = DateTime(
                                          _startDate.year,
                                          _startDate.month,
                                          _startDate.day,
                                          _startTime.hour,
                                          _startTime.minute,
                                          0);
                                      _endDate = _startDate.add(difference);
                                      _endTime = TimeOfDay(
                                          hour: _endDate.hour,
                                          minute: _endDate.minute);
                                    });
                                  }
                                })),
                  ])),
          ListTile(
              contentPadding: const EdgeInsets.fromLTRB(5, 2, 5, 2),
              leading: const Text(‘’),
              title: Row(
                  crossAxisAlignment: CrossAxisAlignment.center,
                  children: <Widget>[
                    Expanded(
                      flex: 7,
                      child: GestureDetector(
                          child: Text(
                            DateFormat(‘EEE, MMM dd yyyy’).format(_endDate),
                            textAlign: TextAlign.left,
                          ),
                          onTap: () async {
                            final DateTime date = await showDatePicker(
                              context: context,
                              initialDate: _endDate,
                              firstDate: DateTime(1900),
                              lastDate: DateTime(2100),
                            );
 
                            if (date != null && date != _endDate) {
                              setState(() {
                                final Duration difference =
                                    _endDate.difference(_startDate);
                                _endDate = DateTime(
                                    date.year,
                                    date.month,
                                    date.day,
                                    _endTime.hour,
                                    _endTime.minute,
                                    0);
                                if (_endDate.isBefore(_startDate)) {
                                  _startDate = _endDate.subtract(difference);
                                  _startTime = TimeOfDay(
                                      hour: _startDate.hour,
                                      minute: _startDate.minute);
                                }
                              });
                            }
                          }),
                    ),
                    Expanded(
                        flex: 3,
                        child: _isAllDay
                            ? const Text(‘’)
                            : GestureDetector(
                                child: Text(
                                  DateFormat(‘hh:mm a’).format(_endDate),
                                  textAlign: TextAlign.right,
                                ),
                                onTap: () async {
                                  final TimeOfDay time = await showTimePicker(
                                      context: context,
                                      initialTime: TimeOfDay(
                                          hour: _endTime.hour,
                                          minute: _endTime.minute));
 
                                  if (time != null && time != _endTime) {
                                    setState(() {
                                      _endTime = time;
                                      final Duration difference =
                                          _endDate.difference(_startDate);
                                      _endDate = DateTime(
                                          _endDate.year,
                                          _endDate.month,
                                          _endDate.day,
                                          _endTime.hour,
                                          _endTime.minute,
                                          0);
                                      if (_endDate.isBefore(_startDate)) {
                                        _startDate =
                                            _endDate.subtract(difference);
                                        _startTime = TimeOfDay(
                                            hour: _startDate.hour,
                                            minute: _startDate.minute);
                                      }
                                    });
                                  }
                                })),
                  ])),
          ListTile(
            contentPadding: const EdgeInsets.fromLTRB(5, 2, 5, 2),
            leading: Icon(
              Icons.public,
              color: Colors.black87,
            ),
            title: Text(_timeZoneCollection[_selectedTimeZoneIndex]),
            onTap: () {
              showDialog<Widget>(
                context: context,
                barrierDismissible: true,
                builder: (BuildContext context) {
                  return _TimeZonePicker();
                },
              ).then((dynamic value) => setState(() {}));
            },
          ),
          const Divider(
            height: 1.0,
            thickness: 1,
          ),
          ListTile(
            contentPadding: const EdgeInsets.fromLTRB(5, 2, 5, 2),
            leading: Icon(Icons.lens,
                color: _colorCollection[_selectedColorIndex]),
            title: Text(
              _colorNames[_selectedColorIndex],
            ),
            onTap: () {
              showDialog<Widget>(
                context: context,
                barrierDismissible: true,
                builder: (BuildContext context) {
                  return _ColorPicker();
                },
              ).then((dynamic value) => setState(() {}));
            },
          ),
          const Divider(
            height: 1.0,
            thickness: 1,
          ),
          ListTile(
            contentPadding: const EdgeInsets.all(5),
            leading: Icon(
              Icons.subject,
              color: Colors.black87,
            ),
            title: TextField(
              controller: TextEditingController(text: _notes),
              onChanged: (String value) {
                _notes = value;
              },
              keyboardType: TextInputType.multiline,
              maxLines: null,
              style: TextStyle(
                  fontSize: 18,
                  color: Colors.black87,
                  fontWeight: FontWeight.w400),
              decoration: InputDecoration(
                border: InputBorder.none,
                hintText: ‘Add description’,
              ),
            ),
          ),
          const Divider(
            height: 1.0,
            thickness: 1,
          ),
        ],
      ));
}
```
## Step 5:
`onTap` callback event of the calendar widget will return the tapped element, tapped date and the tapped appointment details and you can edit the appointment and add it to the application.

 
```xml
Void onCalendarTapped(CalendarTapDetails calendarTapDetails) {
  if (calendarTapDetails.targetElement != CalendarElement.calendarCell &&
      calendarTapDetails.targetElement != CalendarElement.appointment) {
    return;
  }
 
  setState(() {
    _selectedAppointment = null;
    _isAllDay = false;
    _selectedColorIndex = 0;
    _selectedTimeZoneIndex = 0;
    _subject = ‘’;
    _notes = ‘’;
    if (_calendarView == CalendarView.month) {
      _calendarView = CalendarView.day;
    } else {
      if (calendarTapDetails.appointments != null &&
          calendarTapDetails.appointments.length == 1) {
        final Meeting meetingDetails = calendarTapDetails.appointments[0];
        _startDate = meetingDetails.from;
        _endDate = meetingDetails.to;
        _isAllDay = meetingDetails.isAllDay;
        _selectedColorIndex =
            _colorCollection.indexOf(meetingDetails.background);
        _selectedTimeZoneIndex = meetingDetails.startTimeZone == ‘’
            ? 0
            : _timeZoneCollection.indexOf(meetingDetails.startTimeZone);
        _subject = meetingDetails.eventName == ‘(No title)’
            ? ‘’
            : meetingDetails.eventName;
        _notes = meetingDetails.description;
        _selectedAppointment = meetingDetails;
      } else {
        final DateTime date = calendarTapDetails.date;
        _startDate = date;
        _endDate = date.add(const Duration(hours: 1));
      }
      _startTime =
          TimeOfDay(hour: _startDate.hour, minute: _startDate.minute);
      _endTime = TimeOfDay(hour: _endDate.hour, minute: _endDate.minute);
      Navigator.push<Widget>(
        context,
        MaterialPageRoute(
            builder: (BuildContext context) => AppointmentEditor()),
      );
    }
  });
}
```

## Step 6:
After editing the editor fields, you can update/delete the existing meeting details or create a new meeting details based on the editor values.

```xml
Widget build([BuildContext context]) {
  return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
          appBar: AppBar(
            title: Text(getTile()),
            backgroundColor: _colorCollection[_selectedColorIndex],
            leading: IconButton(
              icon: const Icon(
                Icons.close,
                color: Colors.white,
              ),
              onPressed: () {
                Navigator.pop(context);
              },
            ),
            actions: <Widget>[
              IconButton(
                  padding: const EdgeInsets.fromLTRB(5, 0, 5, 0),
                  icon: const Icon(
                    Icons.done,
                    color: Colors.white,
                  ),
                  onPressed: () {
                    final List<Meeting> meetings = <Meeting>[];
                    if (_selectedAppointment != null) {
                      _events.appointments.removeAt(
                          _events.appointments.indexOf(_selectedAppointment));
                      _events.notifyListeners(CalendarDataSourceAction.remove,
                          <Meeting>[]..add(_selectedAppointment));
                    }
                    meetings.add(Meeting(
                      from: _startDate,
                      to: _endDate,
                      background: _colorCollection[_selectedColorIndex],
                      startTimeZone: _selectedTimeZoneIndex == 0
                          ? ‘’
                          : _timeZoneCollection[_selectedTimeZoneIndex],
                      endTimeZone: _selectedTimeZoneIndex == 0
                          ? ‘’
                          : _timeZoneCollection[_selectedTimeZoneIndex],
                      description: _notes,
                      isAllDay: _isAllDay,
                      eventName: _subject == '' ? '(No title)' : _subject,
                    ));
 
                    _events.appointments.add(meetings[0]);
 
                    _events.notifyListeners(
                        CalendarDataSourceAction.add, meetings);
                    _selectedAppointment = null;
 
                    Navigator.pop(context);
                  })
            ],
          ),
          body: Padding(
            padding: const EdgeInsets.fromLTRB(5, 5, 5, 5),
            child: Stack(
              children: <Widget>[_getAppointmentEditor(context)],
            ),
          ),
          floatingActionButton: _selectedAppointment == null
              ? const Text('')
              : FloatingActionButton(
                  onPressed: () {
                    if (_selectedAppointment != null) {
                      _events.appointments.removeAt(
                          _events.appointments.indexOf(_selectedAppointment));
                      _events.notifyListeners(CalendarDataSourceAction.remove,
                          <Meeting>[]..add(_selectedAppointment));
                      _selectedAppointment = null;
                      Navigator.pop(context);
                    }
                  },
                  child:
                      const Icon(Icons.delete_outline, color: Colors.white),
                  backgroundColor: Colors.red,
                )));
}
```
 
