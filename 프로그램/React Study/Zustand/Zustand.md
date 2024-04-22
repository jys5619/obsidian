```bash
npm install zustand
```
## store.ts
```tsx
type State = {
  age: number
  firstName: string
  lastName: string
}

type Action = {
  updateFirstName: (firstName: State['firstName']) => void
  updateLastName: (lastName: State['lastName']) => void
  incrementAsync: () => Promise<void>
}

// Create your store, which includes both state and (optionally) actions
const usePersonStore = create<State & Action>((set) => ({
  age: 0,
  firstName: '',
  lastName: '',
  updateFirstName: (firstName) => set(() => ({ firstName: firstName })),
  updateLastName: (lastName) => set(() => ({ lastName: lastName })),
  incrementAsync: async () =>  {
    await new Promise((resolve) => setTimeout(resolve, 1000));
    set((state)) => {{age: state.age + 1}}
  }
}))
```

App.tsx
```tsx
const logAgeCount = () => {
  const age = usePersonStore.getState().age;
  userPersonStore.setState({age: 1})
}

function App() {
  const { count, incrementAsync } = usePersonStore()
  // 이렇게 하나씩 꺼내는게 더 좋은 방식이다.
  const firstName = usePersonStore((state) => state.firstName); 
  const lastName = usePersonStore((state) => state.lastName);
  const updateFirstName = usePersonStore((state) => state.updateFirstName);
  const updateLastName = usePersonStore((state) => state.updateLastName);
  
  return (
    <div>
      <span>{count}</span>
      <button onClick={inc}>one up</button>
    </div>
  )
}
```