---
title: 使用 PHPStorm 的 Live Templates 封装常用代码
date: 2016-11-19 15:11:46
tags:
---

## PHP Template Group

### fore
```
foreach ($ITERABLE$ as $VAR_VALUE$) {
    $END$
}
```

### forek
```
foreach ($ITERABLE$ as $VAR_KEY$ => $VAR_VALUE$) {
    $END$
}
```

### prif
```
private function $NAME$($PARAMETERS$){
    $END$
}
```

### prisf
```
private static function $NAME$($PARAMETERS$){
    $END$
}
```

### prof
```
protected function $NAME$($PARAMETERS$){
    $END$
}
```

### prosf
```
protected static function $NAME$($PARAMETERS$){
    $END$
}
```

### pubf
```
public function $NAME$($PARAMETERS$){
    $END$
}
```

### pubsf
```
public static function $NAME$($PARAMETERS$){
    $END$
}
```

### thr
```
throw new $END$
```

### tryc
```
try {
    $ITERABLE$
} catch (\$Exception$ $e) {
    $END$
}
```

##
