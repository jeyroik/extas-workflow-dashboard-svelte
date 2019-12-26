<script context="module">
export async function jsonRpc(data, success) {
    let response = await fetch(`process.JSON_RPC__ENDPOINT`, {
          method: 'POST',
          body: JSON.stringify(data)
      }).then(r => r.json());
    let error = response.error;

    if (error) {
        M.toast({html: 'Ошибка: ' + error.message, classes: 'red'});
        return false;
    } else {
        success && success(response.result);
    }

    return response;
}
</script>