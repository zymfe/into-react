``` javascript
export type TypeOfMode = number;

export const NoMode = 0b0000; // 0
export const StrictMode = 0b0001; // 1
// TODO: Remove BlockingMode and ConcurrentMode by reading from the root
// tag instead
export const BlockingMode = 0b0010; // 2
export const ConcurrentMode = 0b0100; // 4
export const ProfileMode = 0b1000; // 8
```