<script>
    import { post } from 'utils.js';

    async function getMyData() {
        return await post(`auth/getMyData`).then(r => {
            if (r && r.accounts) {
                r.funds = r.accounts.reduce((funds, account) => funds + account.balance, 0);
            }
            return r;
        });
    }

    async function getTransactions() {
        return await post(`auth/getTransactions`);
    }
    function wordhighlighting(text) {
        if (!text) return '';
        return text.replace(/[aAаА]/g, '<span style="color: orange;">$&</span>');
    }
</script>

{#if process.browser}
    {#await getMyData()}
        Loading...
    {:then my}
        <section>
            <p style="font-size: xx-large">
                {my.name} 
                ({my.id}, {my.firstName} {my.lastName}, {my.email}, {my.permissionLevel})
            </p>
        </section>
        <section>
            My funds
            <p style="font-size: xx-large; color:{my.funds >= 0 ? 'green' : 'red'}">{my.funds}</p>
        </section>

        <section>
            <ul>
                {#each my.accounts as account}
                    <li>{account.number} ({account.name})</li>
                {/each}
            </ul>
        </section>

        <section>
            {#await getTransactions()}
                Loading...
            {:then transactions}
                <table class="table table-striped table-bordered">
                    <thead>
                        <tr>
                            <th>SenderName</th>
                            <th>Amount</th>
                            <th>CreatedAt</th>
                            <th>Status</th>
                            <th>StatusDetail</th>
                            <th>LoggedInUser</th>
                        </tr>
                    </thead>
                    <tbody>
                        {#each transactions as transaction}
                            <tr>
                                <td><b>{@html wordhighlighting(transaction.senderName)}</b><br>{@html wordhighlighting(transaction.explanation)}</td>
                                <td style="color: {transaction.amount >= 0 ? 'green' : 'red'}">{transaction.amount} {transaction.currency}</td>
                                <td>{@html wordhighlighting(transaction.createdAt)}</td>
                                <td><b>{@html wordhighlighting(transaction.status)}</b><br>{@html wordhighlighting(transaction.statusDetail)}</td>
                                <td>{@html wordhighlighting(transaction.statusDetail)}</td>
                                <td>{@html wordhighlighting(transaction.loggedInUser)}</td>
                            </tr>
                        {/each}
                    </tbody>
                </table>
            {/await}
        </section>
    {/await}
{/if}