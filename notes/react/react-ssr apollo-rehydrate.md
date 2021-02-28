# ssr-apollo-rehydrate

```jsx
import {useLazyQuery} from "@apollo/client";
import {useRecoilState} from "recoil";
import {useEffect} from "react";
import {useMemo} from "react";

import {currentAccount as currentAccountAtom} from "@clumsy_chinchilla/atoms";

import fetchSessionQuery from "./fetchSessionQuery.gql";

export default function MaybeAuthenticated ({children}) {
  const [fetchSession, {error, data, loading}] = useLazyQuery(fetchSessionQuery);
  const [currentAccount, setCurrentAccount] = useRecoilState<string>(currentAccountAtom);
  const useIsomorphicEffect = RUNTIME_ENV === "client" ? useEffect : useMemo;

  useIsomorphicEffect(() => {
    if (!data || !data.session || !data.session.id || currentAccount) {
      return;
    }

    setCurrentAccount(data.session.id);
  }, [loading, currentAccount, data, setCurrentAccount]);

  useIsomorphicEffect(() => {
    if (error) {
      return;
    }

    if (loading || currentAccount) {
      return;
    }

    fetchSession();
  }, [fetchSession, error, loading, currentAccount]);

  if (error && error.message !== "unauthenticated") {
    throw error;
  }

  return children;
}
```