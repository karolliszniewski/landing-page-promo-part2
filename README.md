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

Next step use helper:
create 'app/code/LandingPage/Form/Helper/Data.php'

Data.php content:

```php
<?php
namespace LandingPage\Form\Helper;

use Magento\Framework\App\Helper\AbstractHelper;
use Magento\Framework\App\Helper\Context;
use Magento\Framework\App\Config\ScopeConfigInterface;
use Magento\Store\Model\ScopeInterface;

class Data extends AbstractHelper
{
    const XML_PATH_MODULE_ENABLED = 'landingpage_form/general/enable_module';

    protected $scopeConfig;

    public function __construct(
        Context $context,
        ScopeConfigInterface $scopeConfig
    ) {
        parent::__construct($context);
        $this->scopeConfig = $scopeConfig;
    }

    /**
     * Check if the module is enabled
     *
     * @return bool
     */
    public function isModuleEnabled()
    {
        return $this->scopeConfig->isSetFlag(
            self::XML_PATH_MODULE_ENABLED,
            ScopeInterface::SCOPE_STORE
        );
    }
}
```

![image](https://github.com/user-attachments/assets/2ea82d7d-9c0a-456c-a053-0c772bf83b32)


create 'app/code/LandingPage/Form/Helper/Data.php'

```php
<?php
namespace LandingPage\Form\Helper;

use Magento\Framework\App\Helper\AbstractHelper;
use Magento\Framework\App\Helper\Context;
use Magento\Framework\App\Config\ScopeConfigInterface;
use Magento\Store\Model\ScopeInterface;

class Data extends AbstractHelper
{
    const XML_PATH_MODULE_ENABLED = 'landingpage_form/general/enable_module';

    protected $scopeConfig;

    public function __construct(
        Context $context,
        ScopeConfigInterface $scopeConfig
    ) {
        parent::__construct($context);
        $this->scopeConfig = $scopeConfig;
    }

    /**
     * Check if the module is enabled
     *
     * @return bool
     */
    public function isModuleEnabled()
    {
        return $this->scopeConfig->isSetFlag(
            self::XML_PATH_MODULE_ENABLED,
            ScopeInterface::SCOPE_STORE
        );
    }
}
```

