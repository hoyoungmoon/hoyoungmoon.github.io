---
title: "[React Native] CodePush Custom Install UI 구현"
date: 2023-03-20
categories: react-native
---

## CodePush Custom Install UI 구현

### 커스텀 훅 useCodePush 구현

```javascript
import { useRef, useState } from "react";
import CodePush from "react-native-code-push";

const useCodePush = () => {
  const [downloadProgress, setDownloadProgress] = useState(0);
  const [hasUpdate, setUpdate] = useState(false);
  const isTimeout = useRef(false);

  async function checkUpdateAndInstall() {
    const updateData = await CodePush.checkForUpdate();

    if (updateData) {
      setUpdate(true);
      const downloadedData = await updateData.download?.(
        async ({ totalBytes, receivedBytes }) => {
          setDownloadProgress(
            Math.round((receivedBytes / totalBytes) * 10000) / 100
          );
        }
      );

      await downloadedData?.install?.(
        updateData.isMandatory && !isTimeout.current
          ? CodePush.InstallMode.IMMEDIATE
          : CodePush.InstallMode.ON_NEXT_RESTART
      );
    }
  }

  const checkCodePush = async () => {
    try {
      await checkUpdateAndInstall();
    } catch (err) {
      if (hasUpdate) {
        setDownloadProgress(100);
      }
    } finally {
      setUpdate(false);
    }
  };

  return { checkCodePush, downloadProgress, hasUpdate };
};

export default useCodePush;
```

### 네트워크 이슈로 인한 무한로딩 방지

```javascript
const checkCodePush = async () => {
  const TIME_OUT_LIMIT = 20000;
  try {
    await Promise.race([
      checkUpdateAndInstall(),
      new Promise((_, reject) =>
        setTimeout(() => {
          isTimeout.current = true;
          reject(new Error("codepush timeout"));
        }, TIME_OUT_LIMIT)
      ),
    ]);
  } catch (err) {
    if (hasUpdate) {
      setDownloadProgress(100);
    }
  } finally {
    setUpdate(false);
  }
};
```

### 코드푸시가 정상적으로 적용이 되었다는 표시 필요

```javascript
// App.js

import CodePush from "react-native-code-push";
...

const App = () => {
    useEffect(()=>{
        async function initialize(){
            /**
			 * CodePush로 다운로드한 패키지의 적용 성공 여부를 전송합니다
			 */
			await CodePush.notifyAppReady();
            ...
        }
        initialize();
    }, [])

    ...
}

```

### 커스텀 훅을 사용하여 UI 작성

```javascript
import React from "react";
import {View, Text, ActivityIndicator} from "react-native";
import useCodePush from "~/hooks/useCodePush";

const LoadingScreen = () => {
  const { downloadProgress, checkCodePush, hasUpdate } = useCodePush();

  return (
    {
		hasUpdate ? (
			<View style={{ paddingHorizontal: 22, width: "100%" }}>
				<View style={{ flexDirection: "row", justifyContent: "space-between", marginBottom: 5 }}>
					<Text>필수 업데이트를 진행중입니다</Text>
					<Text>{`${downloadProgress} %`}</Text>
				</View>
				<View style={{ height: 5, width: "100%", backgroundColor: Colors.gray100, borderRadius: 3 }}>
					<View
						style={{ height: 5, width: `${downloadProgress}%`, backgroundColor: Colors.primary300, borderRadius: 3 }}
					/>
				</View>
			</View>
		) : (
			<ActivityIndicator color={Colors.primary300} />
		);
	}
  )
};
```
