A Developer's Guide to React 19 Server Components
  React 19 continues to build upon the foundation of Server Components, a powerful feature that changes how we think about
  rendering and data fetching. This guide explains their usage with practical examples.


  1. What are React Server Components (RSCs)?
  React Server Components are components that run exclusively on the server. They are rendered on the server, and the resulting
  HTML is sent directly to the browser.


  Key Benefits:
     Zero Client-Side JavaScript:* By default, RSCs don't send any JavaScript to the client, reducing bundle sizes and improving
  initial page load times.
     Direct Backend Access:* They can directly access server-side resources like databases, file systems, or internal APIs without
  needing a separate API layer.
     Automatic Code Splitting:* They help automatically split code, as any components they import are only bundled and sent to the
  client if they are "Client Components."


  Server Components can be async functions, making data fetching clean and straightforward. However, they cannot use hooks like
  useState or useEffect or handle user interactions like onClick because they have no client-side presence.

  ---

  #### 2. The Role of `"use client"`


  For interactivity, you need Client Components. These are the traditional React components you're used to. To designate a file as 
  a Client Component, you place the "use client" directive at the very top.


  When a Server Component imports a Client Component, the framework knows to bundle the Client Component's JavaScript and send it
  to the browser, where it will hydrate and become interactive.


  Example: Combining Server and Client Components


  Let's create a PostList (Server Component) that fetches data and an interactive LikeButton (Client Component).


  `app/LikeButton.jsx` (Client Component)

  `jsx
  "use client"; // This directive marks it as a Client Component

  import { useState } from 'react';

  export default function LikeButton({ initialLikes }) {
    const [likes, setLikes] = useState(initialLikes);

    const handleClick = () => {
      setLikes(likes + 1);
    };

    return (
      <button onClick={handleClick}>
        Like ({likes})
      </button>
    );
  }
  `


  `app/PostList.jsx` (Server Component)

  `jsx
  import db from './database'; // Pretend this is our database client
  import LikeButton from './LikeButton'; // Importing the Client Component

  export default async function PostList() {
    // Data fetching happens directly on the server.
    const posts = await db.posts.getAll();

    return (
      <ul>
        {posts.map(post => (
          <li key={post.id}>
            <h2>{post.title}</h2>
            <p>{post.content}</p>
            {/* The Server Component renders the Client Component */}
            <LikeButton initialLikes={post.likes} />
          </li>
        ))}
      </ul>
    );
  }
  `
  Explanation:
  *   The PostList fetches data on the server and sends down the rendered HTML.
  *   The server also sends the JavaScript needed for *only* the LikeButton to become interactive in the browser.

  ---

  #### 3. The Role of `"use server"` (Server Actions)


  Server Actions allow Client Components to invoke functions that execute securely on the server. This is the modern way to handle
  data mutations (like form submissions) without manually creating API endpoints.

  Example: Form Submission with a Server Action


  Let's create a form to add a new post by calling a Server Action.


  `app/actions.js` (Server Action File)

  `javascript
  "use server"; // This entire file contains server-side functions

  import db from './database';

  export async function addPost(formData) {
    const title = formData.get('title');
    const content = formData.get('content');

    if (!title || !content) {
      return { error: 'Title and content are required.' };
    }

    await db.posts.create({ title, content });
  }
  `


  `app/NewPostForm.jsx` (Client Component)

  `jsx
  "use client";

  import { useTransition } from 'react';
  import { addPost } from './actions'; // Import the server action

  export default function NewPostForm() {
    const [isPending, startTransition] = useTransition();

    const handleSubmit = async (event) => {
      event.preventDefault();
      const form = event.currentTarget;
      const formData = new FormData(form);

      startTransition(async () => {
        const result = await addPost(formData);
        if (result?.error) {
          alert(result.error);
        } else {
          form.reset();
        }
      });
    };

    return (
      <form onSubmit={handleSubmit}>
        <input type="text" name="title" placeholder="Post Title" required />
        <textarea name="content" placeholder="Post Content" required />
        <button type="submit" disabled={isPending}>
          {isPending ? 'Submitting...' : 'Add Post'}
        </button>
      </form>
    );
  }
  `
  Explanation:
  *   The addPost function is marked with "use server" and runs only on the server, so it can safely access the database.
  *   The NewPostForm client component imports addPost and calls it directly. React handles the networking behind the scenes,
  simplifying data mutations immensely.