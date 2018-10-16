## Splitting subscriber groups and profiles into separate XML configuration files

The general idea is to create one "base" XML configuration that will have all shared profiles/nodes/subscriber groups 
and then create separate XML for each test case with their own subscriber profiles and subscriber groups

### Creating the base XML configuration
- the base configuration will need partial_init specified for the subscriber database
- the subscriber count will have to cover all subscribers. For instance, if you have 1 subscriber in base profile and then 5 tests 
with 1 subscriber each your subscriber count will have to be 1 + 5 x 1 = 6
```XML
<subscriber_database count="6" name="ClientDB">
   <global_subscriber_config>
      <partial_init>true</partial_init>
   </global_subscriber_config>
```

### Creating the test case XML configuration
- the *subscriber_database* section for each test case XML will have to have same database name and same subscriber count otherwise it 
will overwrite the base *subscriber_database*
- you don't need to specify *partial_init* again
```XML
<subscriber_database count="6" name="ClientDB">
```
- you can define additional *subscriber_profiles* and you have to define at least one *subscriber_group*. 
You cannot load subscriber profile without having subscriber group
- you have to specify *start* and *end* for subscriber group, using *count* does not work
- the *start* and *end* has to be unique amongst all test configurations otherwise you will get an error
- the *name* of the subscriber group has to be unique. Pick something that will clearly identify the test case the group is associated with
```XML
 <subscriber_group start="2" end="2" name="R128957">
```
- subscriber group can refer to any profiles that are already sourced: profiles from base configuration, profiles from previous 
test cases or profiles defined in this XML configuration. For example here we have PCC and subscription profiles (both named "Post") from the base configuration and newly defined UDC and SmartProfiles (both named "R128957)
```XML
<group_profiles>
  <pcc_profile>Post</pcc_profile>
  <subscription_profile>Post</subscription_profile>
  <udc_profile>R128957</udc_profile>
  <SmartProfile>R128957</SmartProfile>
</group_profiles>
```
- subscriber group will be ***disabled after sourcing the configuration***. Make sure to enable it before running the test. 
You can either issue the CLI command:
```
spr:spr subscriber_database subscriber_group:R128957 enable::true
```
or add initial command to the end of XML configuration:
```XML
<command>
<spr name="spr">
  <subscriber_database>
    <subscriber_group name="R128957">
      <enable>true</enable>
    </subscriber_group>
  </subscriber_database>
</spr>
</command>
```

### Running the test(s)
1. Source the base XML configuration
2. Source each test case XML configurations and make sure all subscriber groups are enabled 
3. Run the test

### De-provisioning subscriber profiles and groups

- If you need to de-provision subscriber profile set its *delete* attibute to true. For example, this CLI command will delete SmartProfile R128957:
```
spr:spr subscriber_database subscriber_profiles SmartProfile:R128957;delete:true
```
