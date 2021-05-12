# JS Dynamic Import

정적 import

```javascript
import { Get } from '@nestjs/common';
import * as path from 'path';
...
```

동적 import

```javascript
const modulePath = 'some module path';

import(modulePath)
    .then(obj => \<module object\>)
    .catch(err => \<loading error e.g if no such module\>)
```

or

```javascript
const modulePath = 'some module path';

const module = await import(modulePath);
```

**import() 구문은 함수 아님 (Function.prototype을 상속하지 않으므로 `call`이나 `apply` 호출 못 함)**