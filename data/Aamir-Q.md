## Summary

| Severity        | Issue                                        | Instances |
| --------------- | :------------------------------------------- | :-------: |
| [[L-0](#low-0)] | Centralization risk for privileged functions |     1     |

---

## Lows

[L-0] No way to remove an asD from `isASD` mapping <a id="low-0" name="low-0"></a>

There is no way to remove an asD from the mapping in `asdFactory.sol`. It might give users wrong info. Create a function to handle that.

```Javascript
    File: 2023-11-canto/asD/src/asdFactory.sol
```
