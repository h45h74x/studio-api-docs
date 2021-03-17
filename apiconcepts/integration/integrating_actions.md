Integrating actions
=====

Desktop Integration API provides support for third-party developers to integrate actions inside the Trados Studio desktop applications.

Integrating general actions
-----

The following example demonstrates how to create an action into the Trados Studio application which has a general purpose and integrate it into a custom ribbon group (see: Integrating ribbon groups).

# [C#](#tab/tabid-1)
[!code-csharp[CreatingRibbonGroupActions.cs](code_samples/CreatingRibbonGroupActions.cs.cs#L17-27)]
***

Integrating controller actions
-----
The following example demonstrates how to create an action specific to a controller and integrate it into a custom ribbon group (see: Integrating ribbon groups).

# [C#](#tab/tabid-2)
[!code-csharp[CreatingRibbonGroupActions.cs](code_samples/CreatingRibbonGroupActions.cs.cs#L32-L52)]
***