signTypedDataV4Button.addEventListener("click", async function (event) {
  event.preventDefault()

  // eth_signTypedData_v4 parameters. All of these parameters affect the resulting signature.
  const msgParams = JSON.stringify({
    domain: {
      // This defines the network, in this case, Mainnet.
      chainId: 1,
      // Give a user-friendly name to the specific contract you're signing for.
      name: "Ether Mail",
      // Add a verifying contract to make sure you're establishing contracts with the proper entity.
      verifyingContract: "0xCcCCccccCCCCcCCCCCCcCcCccCcCCCcCcccccccC",
      // This identifies the latest version.
      version: "1",
    },

    // This defines the message you're proposing the user to sign, is dapp-specific, and contains
    // anything you want. There are no required fields. Be as explicit as possible when building out
    // the message schema.
    message: {
      contents: "Hello, Bob!",
      attachedMoneyInEth: 4.2,
      from: {
        name: "Cow",
        wallets: [
          "0xCD2a3d9F938E13CD947Ec05AbC7FE734Df8DD826",
          "0xDeaDbeefdEAdbeefdEadbEEFdeadbeEFdEaDbeeF",
        ],
      },
      to: [
        {
          name: "Bob",
          wallets: [
            "0xbBbBBBBbbBBBbbbBbbBbbbbBBbBbbbbBbBbbBBbB",
            "0xB0BdaBea57B0BDABeA57b0bdABEA57b0BDabEa57",
            "0xB0B0b0b0b0b0B000000000000000000000000000",
          ],
        },
      ],
    },
    // This refers to the keys of the following types object.
    primaryType: "Mail",
    types: {
      // This refers to the domain the contract is hosted on.
      EIP712Domain: [
        { name: "name", type: "string" },
        { name: "version", type: "string" },
        { name: "chainId", type: "uint256" },
        { name: "verifyingContract", type: "address" },
      ],
      // Not an EIP712Domain definition.
      Group: [
        { name: "name", type: "string" },
        { name: "members", type: "Person[]" },
      ],
      // Refer to primaryType.
      Mail: [
        { name: "from", type: "Person" },
        { name: "to", type: "Person[]" },
        { name: "contents", type: "string" },
      ],
      // Not an EIP712Domain definition.
      Person: [
        { name: "name", type: "string" },
        { name: "wallets", type: "address[]" },
      ],
    },
  })

  var from = await web3.eth.getAccounts()

  var params = [from[0], msgParams]
  var method = "eth_signTypedData_v4"

  provider // Or window.ethereum if you don't support EIP-6963.
    .sendAsync(
      {
        method,
        params,
        from: from[0],
      },
      function (err, result) {
        if (err) return console.dir(err)
        if (result.error) {
          alert(result.error.message)
        }
        if (result.error) return console.error("ERROR", result)
        console.log("TYPED SIGNED:" + JSON.stringify(result.result))

        const recovered = sigUtil.recoverTypedSignature_v4({
          data: JSON.parse(msgParams),
          sig: result.result,
        })

        if (
          ethUtil.toChecksumAddress(recovered) ===
          ethUtil.toChecksumAddress(from)
        ) {
          alert("Successfully recovered signer as " + from)
        } else {
          alert(
            "Failed to verify signer when comparing " + result + " to " + from
          )
        }
      }
    )
})
