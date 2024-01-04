# gpt
@@ -48,6 +48,13 @@ public function saveArrInFileJson($FileJsonPath, $fileArr){
    }


    /**
     * @param $token
     * @param $Question
     * @param int $historyMessagesId
     * @param string $model - 'gpt-3.5-turbo', 'gpt-4'
     * @return array
     */
    public function getChatGPTAnswer($token, $Question, $historyMessagesId=0, $model = 'gpt-3.5-turbo'){

        if(empty($historyMessagesId)){
@@ -56,47 +63,34 @@ public function getChatGPTAnswer($token, $Question, $historyMessagesId=0, $model

        $historyMessagesFilePath = $this->historyMessagesDir.'/'.$historyMessagesId.'.json';

        $MessagesArr=$this->getArrByFileJson($historyMessagesFilePath);
        $MessagesArr[]=['role' => 'user', 'content' => $Question];
        $FileArr=$this->getArrByFileJson($historyMessagesFilePath);
        if(empty($FileArr['model_id'])){
            $FileArr['model_id'] = $model;
        } else {
            $model = $FileArr['model_id'];
        }
        $FileArr['MessagesArr'][]=['role' => 'user', 'content' => $Question];

        $client = \OpenAI::client($token);

        if($model=='gpt-3.5-turbo'){
        //if($model=='gpt-3.5-turbo'){
            try {
                $response = $client->chat()->create([
                    'model' => 'gpt-3.5-turbo-0613',
                    'messages' => $MessagesArr,
                    'model' => $model,
                    'messages' => $FileArr['MessagesArr'],
                ]);
            } catch (\Exception $e) {
                return ['error' => 1, 'data' => 'Exception: '.  $e->getMessage()];
            }
            $response->toArray();
            if(!empty($response['choices'][0]['message']['content'])){
                $answerContent = $response['choices'][0]['message']['content'];
                $MessagesArr[] = ['role'=>'assistant', 'content'=>$answerContent];
                $this->saveArrInFileJson($historyMessagesFilePath, $MessagesArr);
                $FileArr['MessagesArr'][] = ['role'=>'assistant', 'content'=>$answerContent];
                $this->saveArrInFileJson($historyMessagesFilePath, $FileArr);

                return ['error' => 0, 'data' => 'Success', 'historyMessagesId' => $historyMessagesId, 'answer'=>$answerContent, 'response'=>$response];
            }
        }

        // TODO GPT-4 API waitlist https://openai.com/waitlist/gpt-4-api , SOON
        if($model=='gpt-4'){
            /*$stream = $client->chat()->createStreamed([
                'model' => 'gpt-4',
                'messages' => [
                    ['role' => 'user', 'content' => $message_text],
                ],
            ]);
            $rowsArr=[];
            foreach($stream as $response){
                $rowsArr = $response->choices[0]->toArray();
            }
            echo '<pre>';
            print_r($rowsArr);
            echo '</pre>';*/
        }
        //}

        return ['error' => 1, 'data' => 'Unknown error'];
    }
