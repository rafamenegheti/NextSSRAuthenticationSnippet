This was developed in [Rocketset](https://www.rocketseat.com.br/) classes

## Getting Started

With this code snippet you can implement a quite complete authentication method to your NextJS project

I will try to pass throw all the primary files so i can give a general notion on how you can implement it

It is important to make it clear that it is assuming that your backend is serving you with endpoints with these functions:

- `/api/me` - giving you the an array with the user permissions and an arry with the user roles.
- `/api/sessions` - reciving the credentials and returning you: token, refreshToken, permissions and roles
  
  
## Main Points
  
 #### `Configuring Axios`
We are using Axios for our requests, so first of all it's important to configure it

In the services/api.ts we are exporting a function with the name `setupAPIClient`, it receives `ctx` as parameter in cases when this function is called by the server side. We are using [Nookies](https://www.npmjs.com/package/nookies) to deal with the cookies, so in the top of the function we retrive the cookies with it
```typescript 
  let cookies = parseCookies(ctx); 
  ```
  
  after it we create our axios session passing the token (even if it do not exist yet) to the headers:
  ```typescript 
    const api = axios.create({
    baseURL: "http://localhost:3333",
    headers: {
      Authorization: `Bear ${cookies["nextauth.token"]}`,
    },
  });
  ```
  now we are going to use Axios interceptors so we can get errors before javascript try to handle then ([see more](https://axios-http.com/docs/interceptors)), this is important cause
  if the token expires, we'll have to refresh it, and the refresh can take some time and in the meantime the requests will continue to happen with the 
  invalid token causing problems, so the rest of this code will basically wait for a request thar returns 401(not authorized)
  ```typescript
  if (error.response.status === 401) {
        if (error.response.data?.code === "token.expired"){
 }
 ```
        
 realize that it is doing a verification if `error.response.data.code === "token.expired"`
 the reason for this is that the api return the error type, in your case you may do it diferent
 
 this function that set axios is being imported in `services/apliClient` and being exported as a constant `api`
 
 
 #### `AuthContext.tsx`
In this file we are making the main context, that will serve all the pages with important informations, here we'll have `signIn` and `signOut` functions, besides that, this context provides informations about the user and a boolean `isAuthenticated` in cases that you want to check if the user in authenticated throw the
browser side.

 #### `Authenticating from the server side`
 
 You can find examples in: `dashboard.tsx`, `index.tsx` and `metrics.tsx`

in it is passed a function to GetServerSideProps that will be executed as a higher order function, by default there is 2 of them,
- withSSRGuest: you should use this one if you in the login page or in a page that do not require authentication to be access

- withSSRAuth: you should use in pages that requires that you are authenticated to access

`withSSRAuth`can also receive an object with like this: 
```typescript
export const getServerSideProps = withSSRAuth(async (ctx) => {
    const apiClient = setupAPIClient(ctx);
    const response = await apiClient.get("/me");

  
    return {
      props: {},
    };
  }, {
      permissions: ['metrics.list'],
      roles: ['administrator'],
  });
  ```
  so the page will only be loaded if the user has the proper permissions, otherwise
  he'll be redirected to another page
  
  
#### `Can`
  
if you want to show some content depending on the users permissions you got two option:
- useCan: this hook will recive a object with the permissions and return true or false 
```typescript
const userCanSeeComponent = useCan({ permissions, roles })
```
-Can: this is a component, its children will only be shown if the permissions passed as props matches
```typescript
 <Can roles={["administrator"]} permissions={['metrics.list']}>
      <div>Métricas</div>
 </Can>
     
