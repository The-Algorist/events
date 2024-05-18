# Customer Journey Tracking with InfluxDB

## Summary

This project aims to track customer journey events within an application using cookies and event data, which are then recorded in a time-series fashion and posted to InfluxDB. The structure of the data in InfluxDB follows a well-defined schema, comprising a measurement called `events` with various tags and fields for efficient storage and querying of time-series data.

The tags in the schema include `event_type`, `content_id`, `user_id`, `device_type`, `location`, and `referral_source`, among others. These tags help categorize and filter events based on different criteria. Additionally, fields such as `timestamp`, `page_url`, `purchase_value`, and `button_clicked` provide valuable information about each event.
