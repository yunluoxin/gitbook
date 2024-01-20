# Xcode 编译问题集锦

1. DT_TOOLCHAIN_DIR cannot be used to evaluate LIBRARY_SEARCH_PATHS, use TOOLCHAIN_DIR instead

Xcode 15 后苹果改变了变量，从 `$DT_TOOLCHAIN_DIR` => `$TOOLCHAIN_DIR`，所以需要分别把 Cocoapods 和 项目中的进行更改，其中， Pod 通过下面的脚步解决，如果主工程中有直接变量引用，需要手动替换！ [StackOverFlow](https://stackoverflow.com/questions/77133579/dt-toolchain-dir-cannot-be-used-to-evaluate-library-search-paths-use-toolchain)

```ruby
 post_install do |installer|
    xcode_base_version = `xcodebuild -version | grep 'Xcode' | awk '{print $2}' | cut -d . -f 1`

    installer.pods_project.targets.each do |target|
        target.build_configurations.each do |config|
            # For xcode 15+ only
             if config.base_configuration_reference && Integer(xcode_base_version) >= 15
                xcconfig_path = config.base_configuration_reference.real_path
                xcconfig = File.read(xcconfig_path)
                xcconfig_mod = xcconfig.gsub(/DT_TOOLCHAIN_DIR/, "TOOLCHAIN_DIR")
                File.open(xcconfig_path, "w") { |file| file << xcconfig_mod }
            end
        end
    end
end
```

