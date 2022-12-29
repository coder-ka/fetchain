# http-resp-chain

```ts
import { chain } from "http-resp-chain";

const resp = chain()
    .if200(async (resp) => {
        const html = await resp.text();

        return html
    })
    .if401(() => alert('Unauthorized!!!'))
    .if404(() => alert('Page not found!!!'))
    .if500(() => alert('Internal Server Error!!!'))

fetch('https://example.com').then(resp.handle).then(resp => {
    switch(resp.status) {
        case 200:
            const html = resp.data // string
            console.log(html)
            break;
        case 401:
        case 404:
        case 500:
            resp.data // void
            break;
        default:
            resp.status // never
            break;
    }
})
```

```ts
// in api-client.ts
import { chain } from "httpchain";

export const apiResp = chain()
        .if400(async (resp) => {
            const errors = await resp.json();

            return errors as { code: string }[]
        })
        .if401(() => (onUnauthorized: () => void) => onUnauthorized())

// in fetch-users.ts
import { apiResp } from "./api-client"

export const fetchUsers = () => fetch('/api/users').then(
    apiResp
        .if200(async (resp) => {
            const data = await resp.json();

            return data as User[]
        })
        .handle
)

// in main.ts
import { fetchUsers } from "./fetch-users";

fetchUsers().then(resp => {
    switch(resp.status) {
        case 200:
            const users = resp.data;
            console.log(users);
            break;
        case 400:
            const errors = resp.data;
            console.log(errors);
            break;
        case 401:
            const handleAuthorized = resp.data;
            handleAuthorized(() => location.href = "/login")
            break;
    }
})
```

```ts
import { chain, statusSym } from "httpchain";

export const respWithShim = chain()
    .shim((res) => Object.assign(res, { [statusSym]: res.statusCode }))
```
