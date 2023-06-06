# SAFE Management

The recommendeded way to manage `SAFE`s is through the `GebSAFEManager` contract, which allows a single address to open multiple `SAFE`s.&#x20;

{% hint style="warning" %}
Pyflex does not fully support `GebSAFEManager` and so for now you can only use it to interact directly with `SAFEEngine`. When you interact directly with `SAFEEngine`, you can only open one 'unmanaged' `SAFE` per address.

If you're not using pyflex, we recommend you to manage your `SAFE`s through the [app](https://app.reflexer.finance) or with [geb-js](https://docs.reflexer.finance/geb-js/).
{% endhint %}

The following sections show you how to manage SAFEs without the GebSAFEManager and thus interact with the `SAFEEngine` contract directly.

&#x20;\[SAFE MANAGER LINK}
