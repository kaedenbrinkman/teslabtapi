# InformationRequestType
Option|Description
-|-
INFORMATION_REQUEST_TYPE_GET_STATUS|Returns a [`VehicleStatus`](../other/vehstatus) message
INFORMATION_REQUEST_TYPE_GET_TOKEN|Returns the vehicle's token, ...
INFORMATION_REQUEST_TYPE_GET_COUNTER|Returns the current crypto counter, can't seem to get it to work, don't see a point
INFORMATION_REQUEST_TYPE_GET_EPHEMERAL_PUBLIC_KEY|Returns the vehicle's ephemeral key, unknown expiration time
INFORMATION_REQUEST_TYPE_GET_SESSION_DATA|Returns a [`SessionInfo`](../other/sessioninfo) message
INFORMATION_REQUEST_TYPE_GET_WHITELIST_INFO|Returns a [`WhitelistInfo`](../other/wlinfo) message
INFORMATION_REQUEST_TYPE_GET_WHITELIST_ENTRY_INFO|Returns a [`WhitelistEntryInfo`](../other/wlentryinfo) message
INFORMATION_REQUEST_TYPE_GET_VEHICLE_INFO|Returns a [`VehicleInfo`](../other/vehinfo) message, but I couldn't get it to work, the vehicle normally sends info automatically when needed, so not a problem
INFORMATION_REQUEST_TYPE_GET_KEYSTATUS_INFO|Returns a [`KeyStatusInfo`](../other/kstatusinfo) message
INFORMATION_REQUEST_TYPE_GET_ACTIVE_KEY|Returns a [`ActiveKey`](../other/activek) message
INFORMATION_REQUEST_TYPE_GET_CAPABILITIES|Returns a [`Capabilities`](../other/capabilities) message
