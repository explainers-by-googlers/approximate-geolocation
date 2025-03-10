# Approximate Geolocation - Permission states and transitions

The `GeolocationPermissionDescriptor` will contain a new `accuracyMode` aspect:

```webidl
dictionary GeolocationPermissionDescriptor : PermissionDescriptor {
  AccuracyMode accuracyMode = "default";
}

enum AccuracyMode {
  "default",
  "high",
  "approximate"
}
```

with the constraint that `"high"` is [stronger
than](https://w3c.github.io/permissions/#ref-for-dfn-stronger-than-1)
`"default"`, which is stronger than `"approximate"`.

In particular, if `"approximate"` is `"denied"` then also the other ones will be
`"denied"`, while  if `"high"` is `"granted"`  then also the other ones will be
`"granted"`.

Moreover, when `"high"` is `"denied"` (for example, this could also happen when
access to precise location has been blocked via Permissions Policy) then
`"default"` will behave exactly as `"approximate"`, so there are seven possible
valid states for the `"geolocation"` permission:

|    | `approximate` | `default` | `high`    |
|----|:-------------:|:---------:|:---------:|
| 1. | `denied`      | `denied`  | `denied`  |
| 2. | `prompt`      | `prompt`  | `denied`  |
| 3. | `allow`       | `allow`   | `denied`  |
| 4. | `prompt`      | `prompt`  | `prompt`  |
| 5. | `allow`       | `prompt`  | `prompt`  |
| 6. | `allow`       | `allow`   | `prompt`  |
| 7. | `allow`       | `allow`   | `allow`   |

Transitions between those states are summarized in the following table:

| Initial state | Website requests `"approximate"`                                                     | Website requests `"default"`                                                                                                                   | Website requests `"high"`                                                                                                                      |
|---------------|--------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| 1.            | No prompt, website gets `PERMISSION_DENIED`                                          | No prompt, website gets `PERMISSION_DENIED`                                                                                                    | No prompt, website gets `PERMISSION_DENIED`                                                                                                    |
| 2.            | Prompt for approximate only and transition to 3. (if granted) or 1. (if denied).     | Prompt for approximate only and transition to 3. (if granted) or 1. (if denied).                                                               | No prompt, website gets `PERMISSION_DENIED`                                                                                                    |
| 3.            | Return approximate location.                                                         | Return approximate location.                                                                                                                   | No prompt, website gets `PERMISSION_DENIED`                                                                                                    |
| 4.            | Prompt for approximate location and transition to 5. if granted. or 1. (if denied).  | Prompt for either approximate or precise location and transition to 6. (if granted approximate) and 7. (if granted precise) or 1. (if denied). | Prompt for either approximate or precise location and transition to 3. (if granted approximate) and 7. (if granted precise) or 1. (if denied). |
| 5.            | Return approximate location.                                                         | Prompt to upgrade from approximate to precise location and transition to 7. (if granted) or 3. (if denied).                                    | Prompt to upgrade from approximate to precise location and transition to 7. (if granted) or 3. (if denied).                                    |
| 6.            | Return approximate location.                                                         | Return approximate location.                                                                                                                   | Prompt to upgrade from approximate to precise location and transition to 7. (if granted) or 3. (if denied).                                    |
| 7.            | Return approximate location.                                                         | Return precise location.                                                                                                                       | Return precise location.                                                                                                                       |

and in the following diagram:

```mermaid
graph TD;
    subgraph Legend
      direction LR
      start1[ ] -.->|website requests approx.| stop1[ ]
      style start1 height:0px;
      style stop1 height:0px;
      start2[ ] -->|website requests default| stop2[ ]
      style start2 height:0px;
      style stop2 height:0px;
      start3[ ] ==>|website requests precise| stop3[ ]
      style start3 height:0px;
      style stop3 height:0px;
      start4[ ] -->|user grants approx.| stop4[ ]
      style start4 height:0px;
      style stop4 height:0px;
      start5[ ] -->|user grants default| stop5[ ]
      style start5 height:0px;
      style stop5 height:0px;
      start6[ ] -->|user denies| stop6[ ]
      style start6 height:0px;
      style stop6 height:0px;
      linkStyle 3 stroke:blue;
      linkStyle 4 stroke:green;
      linkStyle 5 stroke:red;
    end
    1[1<br>approximate: denied,<br>default: denied,<br>high: denied];
    2[2<br>approximate: prompt,<br>default: prompt,<br>high: denied];
    3[3<br>approximate: allow, <br>default: allow, <br>high: denied];
    4[4<br>approximate: prompt,<br>default: prompt,<br>high: prompt];
    5[5<br>approximate: allow, <br>default: prompt,<br>high: prompt];
    6[6<br>approximate: allow, <br>default: allow, <br>high: prompt];
    7[7<br>approximate: allow, <br>default: allow, <br>high: allow];
    4-.->1;
    4-->1;
    4==>1;
    4==>3;
    4-.->5;
    4-->6;
    4-->7;
    4==>7;
    5-->7;
    5-->3;
    5==>7;
    5==>3;
    6==>7;
    6==>3;
    2-.->3;
    2-->3;
    2==>3;
    2-.->1;
    2-->1;
    2==>1;
    2~~~6;
    6~~~1;
    linkStyle 6 stroke:red;
    linkStyle 7 stroke:red;
    linkStyle 8 stroke:red;
    linkStyle 9 stroke:blue;
    linkStyle 10 stroke:blue;
    linkStyle 11 stroke:blue;
    linkStyle 12 stroke:green;
    linkStyle 13 Stroke:green;
    linkStyle 14 stroke:green;
    linkStyle 15 stroke:red;
    linkStyle 16 stroke:green;
    linkStyle 17 stroke:red;
    linkStyle 18 stroke:green;
    linkStyle 19 stroke:red;
    linkStyle 20 stroke:blue;
    linkStyle 21 stroke:blue;
    linkStyle 22 stroke:blue;
    linkStyle 23 stroke:red;
    linkStyle 24 stroke:red;
    linkStyle 25 stroke:red;
```
