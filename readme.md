Hardhat Network doesn't support eth_maxPriorityFeePerGas.  Ref: https://github.com/NomicFoundation/hardhat/issues/1664.
I don't know if they ever decide to support it.

For the time being, I have figured out a workaround to use this method with the hardhat network. You can try this out if your workflow depends upon it and there is no other way to do it or if it requires a lot of work to change the workflow.

1. Edit the following file in your node_modules directory:

    `node_modules/hardhat/internal/hardhat-network/provider/modules/eth.js`

2. Search for `_gasPriceAction` function and add the following code after that function.

    ```js

      _maxPriorityFeePerGasParams(params) {
        return (0, validation_1.validateParams)(params);
      }

      async _maxPriorityFeePerGasAction() {
        return (0, base_types_1.numberToRpcQuantity)(await this._node.getMaxPriorityFeePerGas());
      }

    ```
    > The function `getMaxPriorityFeePerGas` exists in the file `node_modules/hardhat/internal/hardhat-network/provider/node.js` that returns a Hardcoded value of 1gwei. If you want to change that value, you can change it there.

3. Search for `case "eth_gasPrice":` and add the following case after that.

    ```js

      case "eth_maxPriorityFeePerGas":
        return this._maxPriorityFeePerGasAction(...this._maxPriorityFeePerGasParams(params));

    ```

4. Now, you cannot use `npx hardhat` command to run your modified hardhat package because it will run the original binary installed with npm. However, you can use the following commands to run the modified code.

    ```bash

      export hardhat="node node_modules/hardhat/internal/cli/bootstrap.js"

      $hardhat

    ```
    > You need to run the above command everytime your terminal is restarted or you can add it to your `.bashrc` (ubuntu) file for a permanent solution.

**Note: All of your changes will be lost if you update your hardhat package.**

I request the hardhat team to add this feature in the next release. I hope this helps someone.