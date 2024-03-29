# portRange
A simple `awk` program that convert the `portRange` notation to the standard k8s `port` syntax used in kubernetes `networkpolicies.yml` files.

# What is the problem we're trying to solve?
- K8s network policies can get quite lengthy and difficult to create when one is striving to include a large sequence of ingress and/or egress ports. 
- The syntax `portRange: <from-port-#>:<to-port-#>` allows for a more concise yaml file, and will hopefully be adopted in the near future.
- You can use the `portRange` notation in your `networkpolicy_input.yaml` file and then process it using `portRange-2-ports.awk`.
- The output will be a properly formatted `networkpolicy.yaml` file where the `portRange` notation has been converted into a sequence of inidividual `port` entries.

```
ports:                                                ports:
- protocol: TCP             ---> becomes --->         - protocol: TCP
  portRange: 5978:5980                                  port: 5978
                                                      - protocol: TCP
                                                        port: 5979
                                                      - protocol: TCP
                                                        port: 5980
```         

# How to use it?

1. Clone this repo to your local machine or CI/CD server where the script will run. In the example below, I'm using a Linux based system:

```
git clone https://github.com/rm511130/portRange
cd portRange
```

2. The `portRange` repo comes with an example. Let's take a look at it.

```
cat networkpolicy_input.yaml
```

Note that we are using `portRange` twice in the `yaml` file: 
- once for ingress: `portRange: 6000:6005`
- once for egress:  `portRange: 7000:7010`

3. To convert the `networkpolicy_input.yaml` into a properly formatted `networkpolicy.yaml` file you need to execute the following command:

```
awk -f portRange-2-ports.awk networkpolicy_input.yaml > networkpolicy.yaml
```

4. You are now in a position to implement your network policy. Use, for example, the following command:

```
kubectl apply -f networkpolicy.yaml 
```
The expected output should be:
```
networkpolicy.networking.k8s.io/test-network-policy created
```

You can check your new network policy using the following command:

```
kubectl describe networkpolicies.networking.k8s.io
```

# Limitations

If you edit the `networkpolicy_input.yaml` example and change the `portRange` numbers to:

- ingress: `portRange: 7000:13023`
- egress:  `portRange: 4000:6316`

The commands shown below will produce an error:

```
awk -f portRange-2-ports.awk networkpolicy_input.yaml > networkpolicy.yaml
kubectl apply -f networkpolicy.yaml 
```

The error message should look like this:

```
The NetworkPolicy "test-network-policy" is invalid: metadata.annotations: Too long: must have at most 262144 characters
```

If you edit the `networkpolicy_input.yaml` example and change the `portRange` numbers to:

- ingress: `portRange: 7000:13022`
- egress:  `portRange: 4000:6316`

The network policy will work.

Conclusions: 
- There _is_ a limit to how many ports can be included in a network policy file. 
- Consequently, there is a limit to how many ports you can include when using the `portRange` notation.
- Port numbers with more digits consume the available characters at a faster pace than port numbers with fewer digits.


# Network Performance

No negative performance issues have been observed when using very long network policy yaml files.

# When will K8s address the port ranges requirement?

- As of 03/18/2021, the next release of Open Source [Kubernetes v1.21](https://www.kubernetes.dev/resources/release/) will include an alpha-release solution for port ranges.
- See cell D-42 on the “Dashboard” tab of this [Google Sheets document](https://docs.google.com/spreadsheets/d/1I_ybuEI6gNNPJT7xcO_y-eu_BpayTdBX3nerVsHybNA/edit#gid=936265414).
- Issue description: https://github.com/kubernetes/enhancements/issues/2079
- Design details: https://github.com/kubernetes/enhancements/tree/master/keps/sig-network/2079-network-policy-port-range  








