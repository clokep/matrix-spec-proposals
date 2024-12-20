# MSCxxxx: Bulk Updating User Profile API

It can be useful to update more than one field in a profile at a time, this is
especially useful for bridges when initially or periodically syncing users

## Proposal

This proposes adding the `PATCH` and `PUT` endpoints to the current bulk-get `/profile`
endpoint.

**Note**: Clients are encouraged to manipulate fields individually to avoid race conditions.
However, this method allows for bulk updates when needed (e.g. bots managing multiple accounts).

### Partially Update Profile Fields

- **Endpoint**: `PATCH /_matrix/client/v3/profile/{userId}`
- **Description**: Merge the provided JSON object into the user's profile, updating any
  specified keys without altering others, if permitted by the homeserver.
- **Request Body**:

```json
{
  "avatar_url": "mxc://matrix.org/MyNewAvatar",
  "displayname": "John Doe",
  "m.example_field": "new_value1"
}
```

Note that this does not do a [RFC7396](https://datatracker.ietf.org/doc/html/rfc7396) JSON Merge Patch;
it only applies to top-level fields.

### Replace Profile Fields

- **Endpoint**: `PUT /_matrix/client/v3/profile/{userId}`
- **Description**: Replace the entire user's profile with the provided JSON object,
  adding or updating keys and removing any absent ones, if permitted by the homeserver.
- **Request Body**:

```json
{
  "avatar_url": "mxc://matrix.org/MyNewAvatar",
  "displayname": "John Doe",
  "m.example_field": "new_value1"
}
```

## Error Handling

Errors are essentially the same as those defined for the single-field profile
update fields. See the [error handling section](https://github.com/tcpipuk/matrix-spec-proposals/blob/main/proposals/4133-extended-profiles.md#error-handling)
in [MSC4133](https://github.com/matrix-org/matrix-spec-proposals/pull/4133).

## Propagation of Profile Fields

A bulk update which includes `avatar_url` or `displayname` will continue to trigger state events in each
room *for those fields only*. These fields are replicated per-room via member events.

All other fields (unless a future proposal specifies otherwise) WILL NOT trigger state events in
rooms and will exist solely at the global level for storing metadata about the user.

This should slightly reduce propagation of state events if clients are able to
update both fields at once.

## Potential Issues

None foreseen.

## Security Considerations

None foreseen.

## Alternatives

The main alternative to this is only allowing single fields to be updated, as exists
today.

## Unstable Prefixes

Use unstable endpoints when the capability is not yet stable:

  - `/_matrix/client/unstable/uk.tcpip.mscxxxx/profile/{userId}`

### Server capabilities

To identify whether a server supports this MSC, check the server capabilities for:

```json5
{
  "unstable_features": {
    "uk.tcpip.mscxxxx": true,
    // ...
  }
}
```

## Dependencies

Soft depencency on [MSC4133](https://github.com/matrix-org/matrix-spec-proposals/pull/4133)
(it is not very useful without having more than 2 profile fields). This was originally
part of MSC4133 and split out at request of the Spec Core Team (SCT).
