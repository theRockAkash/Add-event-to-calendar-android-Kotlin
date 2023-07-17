# Add-event-to-calendar-android-Kotlin

#Permissions
```
  <uses-permission android:name="android.permission.WRITE_CALENDAR" />
  <uses-permission android:name="android.permission.READ_CALENDAR" />
```

#OnButtonClicked
```
 binding.btAddToCalender.setOnClickListener {
            val result =
                ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_CALENDAR)
            val result1 =
                ContextCompat.checkSelfPermission(this, Manifest.permission.READ_CALENDAR)
            val isGranted =
                result == PackageManager.PERMISSION_GRANTED && result1 == PackageManager.PERMISSION_GRANTED

            if (isGranted) {
                addToCalender(item)
            } else requestPermissionLauncher.launch(
                arrayOf(
                    Manifest.permission.WRITE_CALENDAR,
                    Manifest.permission.READ_CALENDAR
                )
            )

        }
```

#addToCalender()
```
private fun addToCalender(item: EventData?) {
        if (item == null )
            return
        val cr = this.contentResolver
        val cursor: Cursor? = cr.query(
            Uri.parse("content://com.android.calendar/calendars"),
            arrayOf("_id", "calendar_displayName"),
            null,
            null,
            null
        )
        try {
            if (cursor != null && cursor.moveToFirst()) {
                val calNames = arrayOfNulls<String>(if (cursor.count > 3) 3 else cursor.count) //limit to first 3 calendars 
                val calIds = IntArray(if (cursor.count > 3) 3 else cursor.count)
                for (i in calNames.indices) {
                    if (i < 3) {
                        calIds[i] = cursor.getInt(0)
                        calNames[i] = cursor.getString(1)
                        cursor.moveToNext()
                    } else break
                }

                val sdf = SimpleDateFormat("MMMM dd, yyyy HH:mm aa", Locale.ENGLISH)


                val builder = AlertDialog.Builder(this)
                builder.setTitle("Select any one")
                builder.setSingleChoiceItems(
                    calNames, -1
                ) { dialog, which ->
                      addEvent(sdf, item, calIds, which, cr)
                    Utils.toasty(this, "Event added to calendar.")
                    dialog.cancel()
                }
                builder.create().show()
            }
        } catch (e: Exception) {
            e.printStackTrace()
        }
        cursor?.close()
    }

```
#addEvent()

```
private fun addEvent(
        sdf: SimpleDateFormat,
        item: EventData,
        calIds: IntArray,
        which: Int,
        cr: ContentResolver
    ) {
        val startDate: Long = sdf.parse(item.startTime)?.time ?: System.currentTimeMillis()
        val endDate: Long = sdf.parse(item.endTime)?.time ?: System.currentTimeMillis()

        val cv = ContentValues()
        cv.put("calendar_id", calIds[which])
        cv.put("title", item.event)
        cv.put("dtstart", startDate)
        cv.put("hasAlarm", 1)
        cv.put("dtend", endDate)
        cv.put("eventTimezone", TimeZone.getDefault().getID())

        val newEvent: Uri? = cr.insert(Uri.parse("content://com.android.calendar/events"), cv)
        if (newEvent != null) {
            val id = newEvent.lastPathSegment!!.toLong()
            val values = ContentValues()
            values.put("event_id", id)
            values.put("method", 1)
            values.put("minutes", 15) // 15 minutes
            cr.insert(Uri.parse("content://com.android.calendar/reminders"), values)
        }

    }
```
#requestPermissionLauncher
```
    private val requestPermissionLauncher = registerForActivityResult(
        ActivityResultContracts.RequestMultiplePermissions()
    ) { permissions: Map<String, Boolean> ->
        var isGranted = true
        permissions.entries.forEach {
            if (!it.value) {
                isGranted = false
            }
        }
        if (isGranted) {
            addToCalender(viewModel.eventData)
        } else {
            Utils.toasty(this, "Calender permission denied")
        }

    }

```
