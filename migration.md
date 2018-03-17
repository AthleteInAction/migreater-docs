# Zendesk Migration
<sup>For non Zendesk to Zendesk migrations</sup>

### Starter Questions:
1. What system will the client be migrating from?
2. What sort of items do the client wish to migrate? ([groups](#groups), [organizations](#organizations), [users](#users), [tickets](#tickets), [ticket comments](#ticket-comments), attachments, articles, [ticket fields](#ticket-fields), [user fields](#user-fields)…)
3. What type of data is available?
Does the legacy system have any sort of API for accessing all necessary records?
If easier, can the client export the legacy data to .csv or similar format?
4. Will the client need attachments to be migrated? If so, how can we export each attachment?
5. How will the data be mapped? [See below](#data-mapping)

### Zendesk Migration Breakdown
We start with the end result of importing a single [ticket](#tickets) into Zendesk, then work backwards to import any data which any imported [ticket](#tickets) may depend on. For example, a Zendesk [ticket](#tickets) requires a requester, which is the [user](#users) who created the [ticket](#tickets). So naturally, we must import that [user](#users) before we import that [ticket](#tickets). However, a [user](#users) may belong to an [organization](#organizations). In that case we must import that [user's](#users) [organization](#organizations) before we import the [user](#users). Even further, and [organization](#organizations) can belong to a [group](#goups). So we must import that [group](#goups), then the [organization](#organizations), then the [user](#users), then finally the [ticket](#tickets). This is just a small, simple example of the migration process. There are many other dependencies involved in a migration, but in general, the basic process starts with importing the necessary [groups](#goups) first, followed by [organizations](#organizations), then [users](#users), [group memberships](#group-memberships), [organization memberships](#organization-memberships), then finally [tickets](#tickets), [ticket comments](#ticket-comments), and attachments.

### Data Mapping
Data mapping can be the trickiest part a a migration. The better understanding the client has of how Zendesk data is linked together, the easier mapping the data from their legacy system will be. Here is a basic breakdown of Zendesk items with their fields, starting from the top level down:

<sup>**Fields marked with * are required**</sup>

### Ticket Comments
|Field Name|Type|Description|
|:---|:---:|:---|
|[**ticket_id***](#tickets)|text|The external_id of the ticket this comment is associated with|
|**body***|text|The comment text|
|html_body|text|The comment formatted as HTML|
|public|yes or no|yes if a public comment; no if an internal note. The initial value set on ticket creation persists for any additional comment unless you change it|
|[**author***](#users)|email of [user](#users)|The email of the comment author|
|created_at|text|The time the comment was created. Example: "2018-03-17T06:59:22+00:00"|

### Tickets
|Field Name|Type|Description|
|:---|:---:|:---|
|**external_id***|text|A unique identifier linking this ticket to the legacy system's ticket|
|type|text|The type of this ticket. Possible values: "problem", "incident", "question" or "task"|
|subject|text|The value of the subject field for this ticket|
|**description***|text|The first comment on the ticket|
|priority|text|The urgency with which the ticket should be addressed. Possible values: "urgent", "high", "normal", "low"|
|status|text|The state of the ticket. Possible values: "new", "open", "pending", "hold", "solved", "closed"|
|[**requester***](#users)|email of [user](#users)|The user who requested this ticket|
|[submitter](#users)|email of [user](#users)|The user who submitted the ticket. The submitter always becomes the author of the first comment on the ticket|
|[assignee](#users)|email of [user](#users)|The agent currently assigned to the ticket|
|[organization](#organizations)|name of [organization](#organizations)|The organization of the requester. You can only specify the ID of an organization associated with the requester. See Organization Memberships|
|[group](#groups)|name of [group](#groups)|The group this ticket is assigned to|
|collaborators|comma separated list|The emails of users currently cc'ed on the ticket|
|collaborators|comma separated list|The emails of users to add as cc's when creating a ticket|
|follower|comma separated list|The emails of agents currently following the ticket|
|due_at|text|If this is a ticket of type "task" it has a due date. Due date format uses ISO 8601 format.|
|tags|comma separated list|The array of tags applied to this ticket|
|created_at|text|When this record was created. Example: "2018-03-17T06:59:22+00:00"|

### Ticket Fields
|Field Name|Type|Description|
|:---|:---:|:---|
|**type***|text|The type of the ticket field: "checkbox", "date", "decimal", "integer", "regexp", "tagger", "text", or "textarea". Type is not editable once created.|
|**title***|text|The title of the ticket field|
|raw_title|text|The dynamic content placeholder, if present, or the "title" value, if not. See Dynamic Content|
|description|text|The description of the purpose of this ticket field, shown to users|
|raw_description|text|The dynamic content placeholder, if present, or the "description" value, if not. See Dynamic Content|
|position|integer|A relative position for the ticket fields that determines the order of ticket fields on a ticket. Note that positions 0 to 7 are reserved for system fields|
|active|yes or no|Whether this field is available|
|required|yes or no|If it's required for this field to have a value when updated by agents|
|collapsed_for_agents|yes or no|If this field should be shown to agents by default or be hidden alongside infrequently used fields. Classic interface only|
|regexp_for_validation|text|Regular expression field only. The validation pattern for a field value to be deemed valid.|
|title_in_portal|text|The title of the ticket field when shown to end users|
|raw_title_in_portal|text|The dynamic content placeholder, if present, or the "title_in_portal" value, if not. See Dynamic Content|
|visible_in_portal|yes or no|Whether this field is available to end users|
|editable_in_portal|yes or no|Whether this field is editable by end users|
|required_in_portal|yes or no|If it's required for this field to have a value when updated by end users|
|tag|text|A tag value to set for checkbox fields when checked|
|**custom_field_options***|comma separated list|A list of field values. ***Required only for a ticket field of type "tagger"**|
|sub_type_id|integer|"priority" and "status" fields only, inaccessible to all other field types. Defaults to 0. A "priority" sub type of 1 removes the "Low" and "Urgent" options. A "status" sub type of 1 adds the "On-Hold" option.|

### Users
|Field Name|Type|Description|
|:---|:---:|:---|
|**email***|text|The user's primary email address. Writeable on create only. On update, a secondary email is added|
|**name***|text|The users name|
|alias|text|An alias displayed to end users|
|custom_role|text|A custom role if the user is an agent on the Enterprise plan|
|details|text|Any details you want to store about the user, such as an address|
|external_id|text|A unique identifier from another system. The API treats the id as case insensitive. Example: ian1 and Ian1 are the same user|
|moderator|yes or no|Designates whether the user has forum moderation capabilities|
|notes|text|Any notes you want to store about the user|
|only_private_comments|yes or no|yes if the user can only create private comments|
|[organization](#organizations)|name of [organization](#organizations)|The id of the organization the user is associated with|
|[default_group](#groups)|name of [group](#groups)|The user's default group|
|phone|text|The user's primary phone number|
|restricted_agent|yes or no|If the agent has any restrictions; no for admins and unrestricted agents, yes for other agents|
|role|text|The user's role. Possible values are "end-user", "agent", or "admin"|
|signature|text|The user's signature. Only agents and admins can have signatures|
|suspended|yes or no|If the agent is suspended. Tickets from suspended users are also suspended, and these users cannot sign in to the end user portal|
|tags|comma separated list|The user's tags. Only present if your account has user tagging enabled|
|ticket_restriction|text|Specifies which tickets the user has access to. Possible values are: "organization", "groups", "assigned", "requested", null|
|time_zone|text|The user's time zone. Example: “Pacific Time (US & Canada)”|
|verified|yes or no|The user's primary identity is verified or not|

### User Fields
|Field Name|Type|Description|
|:---|:---:|:---|
|**key***|string|create	A unique key that identifies this custom field. This is used for updating the field and referencing in placeholders.|
|**type***|string|Type of the custom field: "checkbox", "date", "decimal", "dropdown", "integer", "regexp", "text", or "textarea"|
|**title***|string|The title of the custom field|
|raw_title|string|The dynamic content placeholder, if present, or the "title" value, if not. See Dynamic Content|
|description|string|User-defined description of this field's purpose|
|raw_description|string|The dynamic content placeholder, if present, or the "description" value, if not. See Dynamic Content|
|position|integer|Ordering of the field relative to other fields|
|active|yes or no|If yes, this field is available for use|
|system|yes or no|If yes, only active and position values of this field can be changed|
|regexp_for_validation|string|Regular expression field only. The validation pattern for a field value to be deemed valid.|
|tag|string|Optional for custom field of type "checkbox"; not presented otherwise.|
|**custom_field_options***|comma separated list|***Required only for a custom field of type "dropdown"**|

### Organizations
|Field Name|Type|Description|
|:---|:---:|:---|
|external_id|text|A unique external id to associate organizations to an external record. During migrations we typically populate this field with the unique id from whatever system we are migrating from|
|**name***|text|A unique name for the organization|
|domain_names|comma separated list|A list of domain names associated with the organization|
|details|text|Any details about the organization, such as the address|
|notes|text|Any notes you have about the organization|
|[group](#groups)|name of [group](#groups)|When a new ticket is created from a user in this organization, this ticket will automatically be put into this group|
|shared_tickets|yes or no|End users in this organization are able to see each other's tickets|
|shared_comments|yes or no|End users in this organization are able to see each other's comments on tickets|
|tags|comma separated list|A list of tags for this organization|

### Organization Memberships
|Field Name|Type|Description|
|:---|:---:|:---|
|**user***|email of [user](#users)|The user for this group membership|
|[**organization***](#organizations)|name of [organization](#organizations)|The organization for this membership|
|default|yes or no|Denotes whether this is the default organization membership for the user

### Groups
|Field Name|Type|Description|
|:---|:---:|:---|
|**name***|text|The name of the group. Can have multiple groups with the same name|

### Group Memberships
|Field Name|Type|Description|
|:---|:---:|:---|
|**user***|email of [user](#users)|The agent for this group membership|
|[**group***](#groups)|name of [group](#groups)|The group for this membership|
|default|yes or no|If yes, tickets assigned directly to the agent will assume this membership's group|
