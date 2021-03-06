## Details

* Title: **PCL-RFC-0002: Better type for indices**
* Author: [kunaltyagi]
* Gitter room: [PointCloudLibrary/PCL-RFC-02](https://gitter.im/PointCloudLibrary/PCL-RFC-02)
* Past discussion: [#3351](https://github.com/PointCloudLibrary/pcl/issues/3351)
  * People of Interest: [jasjuang], [aPonza], [taketwo], [SergioRAgostinho]
* Status: Work-in-progress

[kunaltyagi]: https://github.com/kunaltyagi
[jasjuang]: https://github.com/jasjuang
[aPonza]: https://github.com/aPonza
[SergioRAgostinho]: https://github.com/SergioRAgostinho
[taketwo]: https://github.com/taketwo

## Motivation
Currently, there's mixed use of `int`, `long`, `unsigned int`, `unsigned long`, `unsigned long long` and `std::size_t` for indices. Some effort has been made to unify towards `std::size_t` to reduce the limit of 2 billions points due to `int`.

Using `pcl::index_t` as a compile time configurable alias (default: `std::intN_t` or `std::ptrdiff_t`) would make PCL algorithms work with point clouds of any size.

## Implementation Details
* Create a compile time option: `PCL_LARGE_INDICES`
* Conditionally choose between small or large indices:
    ```cpp
    #ifdef PCL_LARGE_INDICES
    using index_t = std::ptrdiff_t;
    #else
    using index_t = std::int32_t;
    #endif
    ```
* Maybe allow the user to choose smaller indices (32k for int16) by using `PCL_INDEX_SIZE` variable as the length (8, 16, 32, 64, etc.) Don't know if anyone ever desires this, but it's an easy extension.
* Maybe allow the user to choose signed/unsigned indices (double the amount, eg: 64k, 4.2 million) without needing larger datatype

## Pros
* Follows a best practice (as defined in the (Guideline Support Library aka [GSL](https://github.com/microsoft/GSL/))
* Potentially allows operations on actually limitless points (if using 64 bit systems)
* Allow the user to choose a custom/larger platform independent/dependent index size

## Cons
* One of the following complication (IFF we use use `GSL`'s `index_t`):
  * Extra build-time dependency on internet
  * Copy-pasted source file
  * `git` sub-tree/sub-module
* Wastes memory: Largest RAM supported is ~32TB (2^48). That's a clear 25% loss of efficiency for extreme cases, and larger for average use-case. Though this is negated by the explicit choice made by user before compilation.

## ABI/API Breakage
* None for existing internal migration
* Major API breakage for changing `Indices` from `std::vector<int>` to `std::vector<index_t>`

## Effort Required
Minor to medium, split over multiple phases:
1. Internal API modification is underway, would need modification from `std::size_t` to `pcl::index_t`
2. Modification of public API will be faster since it is easily identifiable (function declaration, data members)

## Migration Path
Multi-step migration:
1. Replace `std::vector<int>` with `IndicesVec`
2. Replace `std::size_t` with `pcl::index_t`
3. Replace `IndicesVec = std::vector<int>` with `IndicesVec = std::vector<pcl::index_t>`