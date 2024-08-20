# Front End system design - Crayon

## Packages and libraries
We will use established packages from npm. This list is a rough overview, more details below about some of those below

- React Router Dom 
- Redux Toolkit + Query
- MUI Material (for styling)
- MUI X Grid (table component)
- date-fns (date formating)
- React Testing Library (for testing React components and hooks)
- react i18next (for localisation)
- Vite (for bundling, code splitting, build, dev server)
- Vitest (test client runner that plays nice with Vite)
- Typescript
- Prettier + ESLint
- react-window (for virtualisation used on Activity and Collaboration)
- Zod (for validation, and type inference)
- react-draft-wysiwyg (a Wysiwyg editor for Collaboration)

## Styling library
Wide array of libraries have all listed components available. I will be very opinionated about it as i have most experience with MUI Material, and regardless of the styling library choice, the most popular ones like `Chakra UI`, or `Ant Design` all have built in customisations for styling solutions, so it comes down to preference. 

## Icons
Icons are exclusive to specific designer guidelines. The design from the sketch infers a custom design approach. MUI supports `SvgIconComponent` component which will allow to use custom SVG elements baked into the library if necessary, so we don't need to rely on Material Design icons.

![enter image description here](https://github.com/konstantinB1/crayon-sysdesign/blob/main/Components.png?raw=true) 

## State management
There is wide array of possibilities to choose from when it comes to state management. Modern state management libraries, also have baked in somewhat opinionated way of data fetching concepts, which eases the transition of storing server related data, caching/invalidation, optimistic updates, error handling, async actions, automatic optional retrying, and ways to sync with server side defined schemas. 

We can go either way, but all the benefits mentioned above, and for author's familiarity, and experience i would go with **Redux Toolkit** - `@reduxjs/toolkit` package.
Great thing about RTK is built in support for auto generating endpoints based on backend schema. It has full support for parsing OpenApi3 spec defined schemas, which will in turn generate all the endpoints with full `Typescript` type support for those types. It reduces boilerplate code of having to manually sync server/client data contracts among other things. This approach of course demands that the backend has its endpoints defined in OAS3 spec, and those schemas are exportable, and consumable by RTK cli.

## Component Overview
#### Layout
Wraps the overlay with framework's style providers, global state store, register `SnackbarContext`, and registers routes that will work in browser.  Component should take care of adjusting appropriate width when it comes to responsive design to work on mobile, tablet, desktop, and bigger devices such as TV.

![enter image description here](https://github.com/konstantinB1/crayon-sysdesign/blob/main/Component%20Overview.png?raw=true)


**Components**
##### `<RouterDefs />`
Takes care of all route definitions
##### `<StoreProvider />`
Will encapsulate global state store logic
##### `<SnackbarContext />`
Enable global `Snackbar` component to be triggered programatically via context's defined API. This component will address user's interaction's such as saving, updating data. It will report on pooling request's status for quote lifecycle once it is finished, or failed. The cards themselves will be queued to a specific timer to appear and disappear, and to be minimal as possible not to break UX.
##### `<AboutDeal />`
Contains API network definition for `GET` and `PUT` for changing info about the specific deal (not sure actually what it does). More about network specifics later.

#### React Router Definition Component
Specific library for routing would be **React Router** - `react-router-dom` package. Reason for picking is wide adoption range, and popularity, in the "vanilla" react ecosystems. 
Takes care of route registration. In addition to 3 main routes specific for the domain logic feature, it should also address cases for not matching any route in SPA scenarios with wildcard `*` placeholder. As the component is designed to work on web, the router should use `BrowserRouter` router interface component.
|Route Name| Component |
|--|--|
| `/` | `<GeneralView />` |
| `/activities` | `<ActivitiesView />` |
| `/collaboration` | `<CollaborationView />` |
| `/activities` | `<Activities />` |
| `*` | `<NotFound />` |

## General Tab

### Table component
Table component used is an extension to MUI Material library, and it has built in supports for all the things we need in its free tier. It has built in localisation on the initialisation. The accessibility concerns are all addressed and baked into the library per WCAG standards. Majority of the requirements are built in features, such as column reordering, filtering, localisation. Performance and rendering considerations for this component is that it will be rendered twice, once before the data comes in and we will render placeholders for rows, so we avoid layout shifts. Sorting and ordering is depending on total data availability. Magic number of rows `n` will determine if total amount of rows is below the threshold for letting the sorting, filtering logic be directed to the server, instead the client.

### Input (number/text)
Input components should accomodate inline formatting as user writes the actual prices, the input should localise it depending on currency used. `react-i18next` can be utilised here to support that. Besides React's implementation all major browsers support localisation via Intl global interface, available on `window` object. Alongside MUI's TextField component that will be used for rendering, `zod` library will be utilised to do the client side validation.

### Checkbox
Checkbox for selecting table rows. Pretty straightforward

## Api Design
Table layout should support CRUD operations.

*Note: In this document API definitions will be using JSON Schema, and for readability's, and DRY principles sake i will use `$ref` JSON schema property to call previously defined schemas to refer to some data.*

#### GET

**Endpoint** `/quotes/?offset=[n]&limit=[n]&sortKey=[key]&order-by=[asc|desc]`

**Query Params**
- **Offset** - backend should able to know where to start executing select query
 - **Limit** - how much data should be returned currently based on the `offset + limit`
- **Sort key** - sort by particular key
- **Order by** - ascending or descending

If the total data returned is smaller then `n` number of rows, the sorting aspect can be done on the client side table, no need to call the endpoint. This customisation is available on all major MUI X Grid.

All response types for this endpoint should be - *application/json*

**JSON Schema**

    {
      "type": "object",
       "definitions": {
       "product": {
          "type": "object",
          "properties": {
            "id": {
              "type": ["number", "string"] // Depending on backend DB...
            },
            "price": {
              "type": "number"
            },
            "quantity": {
              "type": "number"
            },
            "ammount": {
              "type": "number"
            },
            "product": {
              "type": "string"
            },
            "deliveryPrice": {
              "type": "number"
            },
            "tax": {
              "type": "number"
            },
            "dicount_ammount": {
              "type": "number"
            },
            "mode": {
              "type": "string",
              "enum: ["preparing", "submitted", "completed"]
            }
          }
        }
      },
      "properties": {
        "status": {
          "type": "number"
        },
        "message": {
          "type": "string"
        },
        "data": {
          "type": "array",
          "items": {
            "$ref": "$/product"
          }
        }
      }
    }

Example successful GET response

    {
      "status": "success"
      "code": 200,
      "data": [
        {
          "id": 21,
          "product": "Product name",
          "quantity": 2,
          "price": 11.50,
          "ammount" 23,
          "mode": "submitted"
        },
        {
          "id":24,
          "product": "Product name 2",
          "quantity": 10,
          "price": 10,
          "ammount" 100,
          "mode": "completed"
        },
        ...
      ]
    }

Example error GET request

    {
      "status": "error"
      "code": 500,
      "details": "An error occured...",
      "data": null
    }
    
**Error handling**
Handling errors returned from the server, should be adjusted on the client side accordingly. If error code is >= 500, we need to provide generic messages, so potentially sensitive data may not leak. If the errors are < 500, for example 403 for permissions, the errors should inform the user about the reason for failure.

**UI Handling**
The table data should be stored in RTK cache, and invalidate once PUT/POST/PATCH requests are dispatched. Data itself is consumed via RTK generated endpoint hook. Table data should have fixed height and before data is fetched using placeholder `Skeleton` table row component until data is fetched. That way, we have much better UX, and we avoid big layout shifts. RTK exposes `isLoading` parameter which can be used to track the API loading state.

#### POST

**Endpoint** `/quotes`

**Body JSON Schema**

    {
       "price": 400,
       "quantity": 2,
       "amount": 900,
       "product": "New product"
    }

Example successful POST request

    {
      "status": 200,
      "message": "success",
      "data": {
        "mode": "submited"
      }
    }

It is debatable will the `amount` field be calculated on the server side. Arguments for having that data immediately available on the UI as we do the optimistic update, against shifting responsibilities on the client can go either way. If not sure about this, its better to do the logic on the server, since it can open a big precedent for a lot of logic to live on the client which should not be the case.

Example error POST request

    {
      "status": "validation_error"
      "code": 400,
      "details": "An error occured...",
      "data": [
        {
          "price": {
            "message": "Must not be empty"
          },
          "quantity": {
            "message: "Must be more than 0 items"
          }
        }
      ]
    }
    
**UI Handling**
UI will handle the tax, and delivery calculation formula and send the total amount to the server. Server SHOULD validate sent request to see if it matches the calculation from the client, and send corresponding error if it doesn't. The reason for moving the logic to the client is better UX, not having to wait for the server to give back the response each time a new item is selected and calculated.
Before data is sent to the server, we should update table column with state data. This is an optimistic update, we expect the server to respond with success. If any error occurs we rollback the update, and display an error message via `Snackbar`. We access the GET endpoint where table data resides, and modify its stored cache, after that request is triggered.
In case data is invalid for any reason, the client would address most common validation fails, but besides the `price` being empty, or accepts contains invalid characters, all other input validation should not be able to be bypassed directly via UI. 
Once POST request is done, GET cache should be invalidated, and refetched **IF** total count is less than `n` number of rows - this is so we get that data sorted appropriately if necessary, if not the sorting will be done on the UI. `Snackbar`component should also verify that the quote was created.

**Error handling**
As mentioned above handling would be done on the client side via `Zod` or some other input validation library. In case user "hacks" its way or tries to POST something from outside allowed origin, the backend would return appropriate validation errors. Mostly this should be intended for UI developers to have better understanding of the API, the end users should not need to know about it, as we want to make UI handle all the validation for UX purposes.

#### PUT

**Endpoint** `/quotes/{id}`

**Path params**
- ID of the field being changed

*If mode is `submited` this API should not be available in the UI*

**Body JSON Schema**

    [
      "items": "array",
      "properties": {
            "price": {
              "type": "number"
            },
            "quantity": {
              "type": "number"
            },
            "ammount": {
              "type": "number"
            },
            "product": {
              "type": "string"
            }
      }
    ]
Example POST request

    [
      {
        "price": 22,
        "product": "Product name",
        "quantity": 3,
        "amount": 66
      },
      {
        ...
      }
    ]

Example successful PUT response

    {
      "status": 200,
      "message": "success",
      "data": {
        "mode": "submited"
      }
    }

Example error PUT request

    {
      "status": "invalid_state_error"
      "code": 400,
      "details": "Unable to edit quotes that are in completed status",
      "data": null
    }
    
**UI Handling**
Data should do the tax, delivery calculation on the client, below the table.
Posts that are in `completed` status will have read only input fields. Either disable the inputs and show that data, or have specific `Typography` component that will replace inputs will plain text HTML elements.

**Error handling**
As shown above if user tries to edit `completed` field the error will occur. The user shouldn't be notified of this, because by default UI will not allow for this option to be available, so no visual indicators should occur.

#### DELETE

**Endpoint** `/quotes`

*If mode is `submited` this API should not be available in the UI*

**Body JSON Schema**

    [
      "items": "array",
      "type": ["number", "string"] // Depending on id type
    ]
Example DELETE request

    [11, 55, 12, 77]

Example successful PUT response

    {
      "status": 204,
      "message": "success"
    }

Example error PUT request

    {
      "status": "internal_error"
      "code": 500,
      "details": "Unknown error",
      "data": null
    }
    
**UI Handling**
Fields that are checked should be deleted optimistically and removed from the UI in same way POST/PUT requests work. Im somewhat unclear if checkbox delete buttons should be deletable if status is `completed`. If no, delete icons should be removed from those.

**Error handling**
Revert the previous state - deleted rows will come back to its original place, and `Snackbar` will provide appropriate error details depending on the response code.

## Activity tab
Requirements dictate real time collaboration so when it comes to network we basically get down to 2 logical choices:

- Web sockets
- Server side events

Main difference between 2, is that web sockets are 2 way communication, while server side events are 1 way - in plain english SSE opens a stream of GET requests, while POST are still relying on plain HTTP, while web sockets are streaming both GET and POST.

Depending on user volume, and hosting characteristics, appropriate network protocol would be determined in the backend. For this specific feature, web sockets would be a better fit since we have to rely on syncing real time 2 way data flow. While RTK does not directly supports web sockets, it can be extended through middleware, and streaming updates available in RTK ecosystem. Streaming updates allow us to establish a Websocket connection to the endpoint, the handshake part is handled via HTTP protocol so the handshaking aspect can be established via RTK query endpoint, and once it is established the web socket connection can be established. Now all the incoming data will be coming into the initial cache entry when the endpoint is created, and processed via native browser `Websocket` object. Responses may vary depending on the action, for the simplicity of this document we will focus only on getting, and posting data.

## Api Design

Establishing a handshake request to web socket protocol in the initial GET request

**Endpoint** `\collab\{quote-id}?content=[string]&userId=[userId]&startTime=[timestamp]&endTime=[timestamp]&offset[n]`

**Path params**
- `quote-id`  - fetches the collaboration data from the server for a particular quote

**Query params**
- `content`  - specific content to be filtered
- `userId` - messages posted by a specific user
- `startTimestamp` - search from
- `endTimeStamp` - search to
- `offset` - a number that will load data, if user scrolls to the end of the page, depending on the last offset message `id`

**JSON Schema**

    {
      "type": "object",
       "definitions": {
       "message": {
          "type": "object",
          "properties": {
            "id": {
              "type": ["number", "string"] // Depending on backend DB...
            },
            "username": {
              "type": "string"
            },
            "userId": {
              "type": ["number", "string"] // Depending on backend DB...
            },
            "dateTimestamp": {
              "type": "number"
            },
            "content": {
              "type": "string"
            },
            "mode": {
              "type": "string",
              "enum: ["edit", "delete", "add"]
            }
          }
        }
      },
      "properties": {
        "status": {
          "type": "number"
        },
        "message": {
          "type": "string"
        },
        "data": {
          "type": "array",
          "items": {
            "$ref": "$/message"
          }
        }
      }
    }

Example successful GET response

    {
      "status": "success"
      "code": 200,
      "data": [
        {
          "id": 21,
          "content": "<p>Hello world</p>",
          "name": "John Doe",
          "dateTimestamp": 38483823,
          "mode": "add"
        },
        {
          "id": 21,
          "content": "Oops sorry",
          "name": "Jane Doe",
          "dateTimestamp": 39483823,
          "mode": "edit"
        },
        ...
      ]
    }


After the data is fetched in the endpoint setup for RTK we open up a websocket connection listening to changes and cache modified depending on the `mode` from the incoming server event.

### The UI
The UI part will primarily use `Timeline` component to display actual user's messages. Data will be virtualised via `react-window`library which will allow us to scroll performantly without creating overhead in the DOM. One thing about `react-window` which will be challenging is make it available to work on all devices, because the library uses absolute positioning in order to create little overhead as possible, because browser does not really recalculate geometry when elements are floating in the layout. Only the viewport elements will be rendered, the rest is just a padding placeholder to create the illusion of scrolling. Therefore `ref` measures will need to be taken on initial render, and listening for the `resize` event if the screen changes so the content can be calculated appropriately for all screen widths. 

When user scrolls on bottom of the page, it will trigger a new network request that will take into consideration current filter settings, and `offset` with last message `id` available on the current scroll. Fast scrolling can trigger multiple requests, it is important to track the timestamp of the sent requests, so we can now if next one finished before previous and avoid race conditions.

On bottom of the screen there will be a fixed container that allows user to post simple text messages. That container can be expanded using `Collapse` component to extend the possibilities of the message, using a wysiwyg editor.

When user leaves a page, web socket subscription channel will unsubscribe.

### Event types
`mode` of the message is an indicator of the nature of a particular incoming data from the server. Authenticated user will have privileges to edit its messages, if the authenticated id is the same as the one incoming from the server event - `userId`  If the id's match user will have additional privileges to edit or delete previous messages. If user decides to edit the message, we will need to go into 

`edit` mode on the route's state. When in that particular mode, scrolling is disabled, and screen is fixed to the message being edited. If user bails out of edit, `Dialog` component will prompt him to confirm he wants to discard edited changes.

    {
      "id": 21,
      "userId": 11,
      "content": "<p>Changed content</p>",
      "name": "John Doe",
      "dateTimestamp": 38483823,
      "mode": "edit"
    }

`delete` mode incoming message event indicates that user with specific `userId` and `id` is confirmed to be deleted, and the message will be replaced with generic text that indicates deletion of that message.

    {
      "id": 21,
      "userId": 11,
      "mode": "delete"
    }

`post` mode is adding a new message on top of the stack queue. If the filter is used while user posts a message, the filtered data should include messages that existed outside of the GET request, which are stored in the same cache entry.

    {
      "id": 21,
      "userId": 11,
      "content": "<p>Changed content</p>",
      "name": "John Doe",
      "dateTimestamp": 38483823,
      "mode": "add"
    }

If user posts a message, the content used will need to be sanitised for potential sql injections, so the data sent to the server is not malicious. This is a built in feature of react-draft- wysiwyg editor.


## Activity Tab
When user access the route, it will immediately call an endpoint to fetch data from the server, of the latest activities in the quotes. Additionally the endpoint will setup a SSE connection to the server, so it can give streaming updates to activities that took place on this particular quote.

Establishing a handshake request to web socket protocol in the initial GET request

**Endpoint** `\collab\{quote-id}?offset=[n]`

**Path params**
- `quote-id`  - fetches the collaboration data from the server for a particular quote

**Query params**

`offset` a number that will load data, if user scrolls to the end of the page, depending on the last offset activity `id`



    {
      "type": "object",
       "definitions": {
       "activities": {
          "type": "object",
          "properties": {
            "id": {
              "type": ["number", "string"] // Depending on backend DB...
            },
            "type": {
              "type": "string",
              "enum": ["created", "changed"]
            },
            "changes": {
              "type": "object",
              "properties": {
                "type": {
                  "type": "string"
                },
                "previous": {
                  "type": "string"
                },
                "current": {
                  "type": "string"
                }
              }
            },
            "documentInfo": {
              "type": "object",
              "properties": {
                "company": {
                  "type": "string"
                },
                "customer": {
                  "type": "string"
                },
                "name": {
                  "type": "string"
                },
                "url": {
                  "type": "string"
                },
                "signUrl": {
                  "type": "string"
                }
              }
            }
          }
        }
      },
      "properties": {
        "status": {
          "type": "number"
        },
        "message": {
          "type": "string"
        },
        "data": {
          "type": "array",
          "items": {
            "$ref": "#/activities"
          }
        }
      }
    }

Example successful GET response

    {
      "status": "success"
      "code": 200,
      "data": [
        {
          "id": 21,
          "type": "created",
          "changes": null,
          "timestamp": 342343233,
          "documentInfo": {
            "id": 55,
            "url": "/some/url/that/contains/document.pdf",
            "signUrl": "/sign/url",
            "name": "New document",
            "customer": "John Connor",
            "company": ""
          }
        },
        {
          "id": 21,
          "type": "changed",
          "changes": {
              "type": "amount",
              "previous": "Old amount",
              "current": "New amount"
           },
          "timestamp": 362343233,
          "documentInfo": null
        },
        ...
      ]
    }



### The UI
The UI is similar to the Collaboration tab, in a sense it uses the same key component - `Timeline`. Data is stored inside RTK cache on the client, and updates - update the cache entry.  The network is greatly simplified in contrast to Collaboration tab, as we don't have filter.
