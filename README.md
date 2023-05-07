# 🏁 Quick start

Before diving into the thick of things, consider editing your `tsconfig.json` file to be able to use absolute instead of relative path.&#x20;

It will prevent you from having to write imports like:

`import { useTranslations } from "../../../../i18n";`&#x20;

```diff
 {
     "target": "es5",
+    "baseUrl": "src"
     //...
 }
```

Start by declaring the text keys you'll need in each component.&#x20;

`src/components/MyComponent.tsx`

```diff
+import { declareComponentKeys } from "i18nifty";

 type Props = {
     name: string;
 };

 export function MyComponent(props: Props) {
     const { name } = props;
     
     return (
         <>
             <h1>Hello {name}</h1>
             <h3>How are you feeling today?</h3>
             <p>
                 Click <a href="https://example.com">hrere</a> to 
                 learn more about this website
             </p>
         </>
     );
 }

+export const { i18n } = declareComponentKeys<
+    | { K: "greating"; P: { who: string; } }
+    | "how are you"
+    | { K: "learn more"; P: { href: string; }; R: JSX.Element }
+>()({ MyComponent });
```

`src/components/MyOtherComponent.tsx`

```diff
+import { declareComponentKeys } from "i18nifty";

 type Props = {
     messageCount: number;
 };

 export function MyOtherComponent(props: Props) {
     const { messageCount } = props;
     
     return (
         <>
             <span>You have {messageCount} unread messages.</span>
             <button>Open</button>
             <button>Delete</button>
         </>
     );
 }

+export const { i18n } = declareComponentKeys<
+    | "open"
+    | "delete"
+    | { K: "unread messages"; P: { howMany: number; } }
+>()({ MyOtherComponent });
```

then create your `src/i18n.tsx` file: &#x20;

```tsx
import { createI18nApi, declareComponentKeys } from "i18nifty";
export { declareComponentKeys };

//List the languages you with to support
export const languages = ["en", "fr"] as const;

//If the user's browser language doesn't match any 
//of the languages above specify the language to fallback to:  
export const fallbackLanguage = "en";

export type Language = typeof languages[number];

export type LocalizedString = Parameters<typeof resolveLocalizedString>[0];

export const { 
	useTranslation, 
	resolveLocalizedString, 
	useLang, 
	$lang,
	useResolveLocalizedString,
	/** For use outside of React */
	getTranslation 
} = createI18nApi<
    | typeof import ("components/MyComponent").i18n
    | typeof import ("components/MyOtherComponent").i18n
>()(
    { languages, fallbackLanguage },
    {
        "en": {
            "MyComponent": {
                "greating": ({ who })=> `Hello ${who}`,
                "how are you": "How are you feeling today?",
                "learn more": ({ href }) => (
                    <>
                        Learn more about 
                        <a href={href}>this website</a>.
                    </>
                )
            },
            "MyOtherComponent": {
                "open": "Open",
                "delete": "Delete",
                "unread messages": ({ howMany })=> {
                    switch(howMany){
                        case 0: return `You don't have any new message`;
                        case 1: return `You have a new message`;
                        default: return `You have ${howMany} new messages`;
                    }
                }
            },
        },
	/* spell-checker: disable */
	"fr": {
            "MyComponent": {
                "greating": ({ who })=> `Bonjour ${who}`,
                "how are you": "Comment vous sentez vous au jour d'hui?",
                "learn more": ({ href }) => (
                    <>
                        En savoir plus à propos de  
                        <a href={href}>ce site web</a>.
                    </>
                )
            },
            "MyOtherComponent": {
                "open": "Ouvrir",
                "delete": "Supprimer",
                //We will translate this later, for now, fallback to english
                "unread messages": undefined
            },
        }
	/* spell-checker: enable */
    }
);
```

<details>

<summary>Extra steps for Next.js</summary>

Edit your `i18n.ts` file like so: &#x20;

```diff
-import { createI18nApi } from "i18nifty";
+import { createI18nApi } from "i18nifty/next";

 export const { 
 	 useTranslation, 
 	 resolveLocalizedString, 
	 useLang, 
	 evtLang,
	 useResolveLocalizedString,
+        withLang
 } = createI18nApi<
```

Create an `pages/_app.tsx` file (if you didn't had one already):

```jsx
import DefaultApp from "next/app";
import { withLang } from "../i18n";

export default withLang(DefaultApp);
```

</details>

Now go back to your component and use the translation function: &#x20;

```diff
+import { useTranslation, declareComponentKeys } from "i18n"; //You can import it like that thanks to baseUrl
   
 type Props = {
     name: string;
 };

 export function MyComponent(props: Props) {
     const { name } = props;
     
+    const { t } = useTranslation({ MyComponent });
     
     return (
         <>
-            <h1>Hello {name}</h1>
+            <h1>{t("greeting", { who: name })}</h1>
-            <h3>How are you feeling today?</h3>
+            <h3>{t("how are you")}</h3>
-            <p>
-                Click <a href="https://example.com">hrere</a> to 
-                learn more about this website
-            </p>
+            <p>{t("learn more", { href: "https://example.com" })}</p>
         </>
     );
 }

 export const { i18n } = declareComponentKeys<
     | { K: "greating"; P: { who: string; } }
     | "how are you"
     | { K: "learn more"; P: { href: string; }; R: JSX.Element }
 >()({ MyComponent });
```

And so forth for your other components.
