###Create the Configuration XML File

#Create the system.xml File in 'app/code/Vendor/Module/etc/adminhtml/system.xml'

```xml
<?xml version="1.0"?>
<!-- app/code/LandingPage/Form/etc/adminhtml/system.xml -->
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Config/etc/system_file.xsd">
    <system>
        <section id="landingpage_form_section" translate="label" sortOrder="10" showInDefault="1" showInWebsite="1" showInStore="1">
            <label>LandingPage Form Configuration</label>
            <tab>general</tab>
            <resource>LandingPage_Form::config</resource>
            <group id="general" translate="label" sortOrder="10" showInDefault="1" showInWebsite="1" showInStore="1">
                <label>General Settings</label>
                <field id="enable_module" translate="label comment" type="select" sortOrder="10" showInDefault="1" showInWebsite="1" showInStore="1">
                    <label>Enable Module</label>
                    <comment>Choose whether to enable or disable the module.</comment>
                    <source_model>Magento\Config\Model\Config\Source\Yesno</source_model>
     
                </field>
            </group>
        </section>
    </system>
</config>

```
