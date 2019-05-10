##NSSecureCoding与NSCoding协议
### NSCoding

- (void)encodeWithCoder:(NSCoder *)aCoder;
- (nullable instancetype)initWithCoder:(NSCoder *)aDecoder; // NS_DESIGNATED_INITIALIZER


### NSSecureCoding 

- @property (class, readonly) BOOL supportsSecureCoding;
